build-node:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.2-node -f Dockerfile.node . --push

build:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.2 -f Dockerfile . --push

build-node-dev	:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.2-node-dev -f Dockerfile.node-dev . --push

build-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.2-alpine -f Dockerfile-alpine . --push

build-node-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.2-node-alpine -f Dockerfile.node-alpine . --push