# Image URL to use all building/pushing image targets
APP_NAME ?= knative-function-controller
IMG ?= $(APP_NAME):latest
IMG-CI = $(DOCKER_PUSH_REPOSITORY)$(DOCKER_PUSH_DIRECTORY)/$(APP_NAME):$(DOCKER_TAG)

all: test manager

# Resolve dependencies
resolve:
	dep ensure -v -vendor-only

.PHONY: pull-licenses
pull-licenses:
ifdef LICENSE_PULLER_PATH
	bash $(LICENSE_PULLER_PATH)
else
	mkdir -p licenses
endif

# Run tests
test: generate fmt vet symlinks
	go test -v ./pkg/... ./cmd/... -coverprofile cover.out

.ONESHELL:
symlinks:
	cd config/crds
	-rm 300*.yaml
	ln -s ../../vendor/github.com/knative/build/config/300-build.yaml 300-build.yaml
	ln -s ../../vendor/github.com/knative/build/config/300-buildtemplate.yaml 300-buildtemplate.yaml
	ln -s ../../vendor/github.com/knative/serving/config/300-revision.yaml 300-revision.yaml
	ln -s ../../vendor/github.com/knative/serving/config/300-route.yaml 300-route.yaml
	ln -s ../../vendor/github.com/knative/serving/config/300-service.yaml 300-service.yaml
	ln -s ../../vendor/github.com/knative/serving/config/300-configuration.yaml 300-configuration.yaml

# Generate code
generate: resolve
	go generate ./pkg/... ./cmd/...

# Run go fmt against code
fmt:
	go fmt ./pkg/... ./cmd/...

# Run go vet against code
vet:
	go vet ./pkg/... ./cmd/...

qt:
	go test -v ./pkg/... ./cmd/... -coverprofile cover.out

# Build manager binary
manager: generate fmt vet
	go build -o bin/manager github.com/kyma-project/kyma/components/knative-function-controller/cmd/manager

# NOTE: does not work anymore until https://github.com/kubernetes-sigs/kubebuilder/issues/400 is implemented
# Run against the configured Kubernetes cluster in ~/.kube/config
run: generate fmt vet
	go run ./cmd/manager/main.go

# Install CRDs into a cluster
install:
	kubectl apply -f config/crds/runtime_v1alpha1_function.yaml

# Generate manifests e.g. CRD, RBAC etc.
manifests:
	go run vendor/sigs.k8s.io/controller-tools/cmd/controller-gen/main.go all

# Deploy controller in the configured Kubernetes cluster in ~/.kube/config
deploy: manifests install
	kubectl apply -f config/crds
	kustomize build config/default | kubectl apply -f -
	kubectl apply -f config/config.yaml

# Build the docker image
.PHONY: docker-build
docker-build: resolve pull-licenses manifests
	docker build . -t $(IMG)
	@echo "updating kustomize image patch file for manager resource"
	sed -i'' -e 's@image: .*@image: '"${IMG}"'@' ./config/default/manager_image_patch_remote_dev.yaml ./config/default/manager_image_patch_local_dev.yaml

# Push the docker image
.PHONY: docker-push
docker-push:
	docker tag $(IMG) $(IMG-CI)
	docker push $(IMG-CI)

.PHONY: ci-pr
ci-pr: docker-build docker-push

.PHONY: ci-master
ci-master: docker-build docker-push

.PHONY: ci-release
ci-release: docker-build docker-push

.PHONY: path-to-referenced-charts
path-to-referenced-charts:
	@echo "resources/knative-function-controller"