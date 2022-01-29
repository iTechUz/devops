.PHONY: show build release

VERSION=$(shell date +%y%m%d%H%M%S)
IMAGE=registry.gitlab.com/bektechno/bektech_frontend/bektech_frontend

build:
	docker build -t $IMAGE:$VERSION .

show:
	echo $VERSION


# show the docker image name and tag
release:
	echo $(IMAGE):$(VERSION)
#:$(VERSION))