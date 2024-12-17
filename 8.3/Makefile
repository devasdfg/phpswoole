build-node:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3-node -f Dockerfile.node . --push

build:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3 -f Dockerfile . --push

build-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3-alpine -f Dockerfile-alpine . --push

build-node-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3-node-alpine -f Dockerfile.node-alpine . --push