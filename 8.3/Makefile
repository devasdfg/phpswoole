build-node:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3-node -f Dockerfile.node . --push

build:
	docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/devasdf/phpswoole:8.3 -f Dockerfile . --push