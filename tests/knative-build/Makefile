APP_NAME = knative-build-acceptance-tests
IMG = $(DOCKER_PUSH_REPOSITORY)$(DOCKER_PUSH_DIRECTORY)/$(APP_NAME)
TAG = $(DOCKER_TAG)

.PHONY: build-image
build-image:
	docker build -t $(APP_NAME):latest .

.PHONY: push-image
push-image:
	docker tag $(APP_NAME) $(IMG):$(TAG)
	docker push $(IMG):$(TAG)

.PHONY: ci-pr ci-master ci-release
ci-pr ci-master ci-release: build-image push-image
