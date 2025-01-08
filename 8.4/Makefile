build-node:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.4-node -f Dockerfile.node . --push

build-node-dev	:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.4-node-dev -f Dockerfile.node-dev . --push

build:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.4 -f Dockerfile . --push

build-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.4-alpine -f Dockerfile-alpine . --push

build-node-alpine:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.4-node-alpine -f Dockerfile.node-alpine . --push