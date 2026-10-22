# PHP Swoole Base Images

[![Build and push images](https://github.com/devasdfg/phpswoole/actions/workflows/build.yml/badge.svg)](https://github.com/devasdfg/phpswoole/actions/workflows/build.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Docker base images for PHP applications running on [Swoole](https://github.com/swoole/swoole-src). They build on the official [`phpswoole/swoole`](https://hub.docker.com/r/phpswoole/swoole) images and add common PHP extensions, a production-tuned `php.ini`, and optional Node.js variants for building frontend assets.

Images are published to Docker Hub as [`devasdf/phpswoole`](https://hub.docker.com/r/devasdf/phpswoole).

## Available Images

All images use **Swoole 6.2**.

| PHP | Debian (amd64, arm64) | Alpine (amd64) |
|-----|-----------------------|----------------|
| 8.5 | `8.5`, `8.5-node`, `8.5-node-dev` | `8.5-alpine`, `8.5-node-alpine` |
| 8.4 | `8.4`, `8.4-node`, `8.4-node-dev` | `8.4-alpine`, `8.4-node-alpine` |
| 8.3 | `8.3`, `8.3-node`, `8.3-node-dev` | `8.3-alpine`, `8.3-node-alpine` |
| 8.2 | `8.2`, `8.2-node`, `8.2-node-dev` | `8.2-alpine`, `8.2-node-alpine` |

### Variants

| Tag suffix | Base | Description |
|------------|------|-------------|
| _(none)_ | `phpswoole/swoole:6.2-phpX.Y` | PHP + Swoole with the extra extensions |
| `-node` | Debian image above | Adds Node.js 22 (via NodeSource) for frontend builds |
| `-node-dev` | Debian image above | Adds Node.js 24 (via NodeSource) plus a Supervisor program that runs `npm run dev` in `/app` |
| `-alpine` | `phpswoole/swoole:6.2-phpX.Y-alpine` | Smaller Alpine-based image with the same extensions |
| `-node-alpine` | Alpine image above | Adds Node.js and npm from the Alpine repositories |

## Features

- Built on the official `phpswoole/swoole` images
- Additional PHP extensions:
  - GD (FreeType, JPEG and WebP support)
  - Intl
  - Zip
  - Gettext
  - BCMath
  - PCNTL
  - Exif
  - MongoDB (via PECL)
- Custom `php.ini` installed at `/usr/local/etc/php/php.ini`
- Working directory set to `/app`
- Multi-arch Debian images (`linux/amd64`, `linux/arm64`)
- Rebuilt and pushed automatically by GitHub Actions on every push to `main`

## Usage

### Base image

```dockerfile
FROM devasdf/phpswoole:8.5

COPY . /app
RUN composer install --no-dev --optimize-autoloader
```

### Alpine image

```dockerfile
FROM devasdf/phpswoole:8.5-alpine

COPY . /app
```

### Node.js variant (frontend builds)

```dockerfile
FROM devasdf/phpswoole:8.5-node

COPY . /app
RUN composer install --no-dev --optimize-autoloader \
    && npm ci \
    && npm run build
```

### Node.js dev variant (local development)

The `-node-dev` image ships a Supervisor config (`/etc/supervisor/conf.d/npm.conf`) that runs `npm run dev` from `/app` and sends its output to the container logs. This works well for a hot-reloading dev server (Vite, for example) next to your Swoole app.

```yaml
# docker-compose.yml
services:
  app:
    image: devasdf/phpswoole:8.5-node-dev
    volumes:
      - .:/app
    ports:
      - "9501:9501"   # Swoole
      - "5173:5173"   # Vite dev server
```

## PHP Configuration

The bundled `php.ini` is based on PHP's production defaults, with these notable changes:

| Setting | Value |
|---------|-------|
| `memory_limit` | `1024M` |
| `max_execution_time` | `0` (unlimited; long-running Swoole workers) |
| `max_input_time` | `120` |
| `max_input_vars` | `10000` |
| `post_max_size` | `300M` |
| `upload_max_filesize` | `300M` |
| `expose_php` | `Off` (no `X-Powered-By` header) |
| `display_errors` | `Off` |
| `log_errors` | `On` |

To override settings, copy your own file over it or add an extra `.ini` file:

```dockerfile
COPY my-overrides.ini /usr/local/etc/php/conf.d/zz-overrides.ini
```

## Repository Layout

```
.
├── .github/workflows/build.yml   # CI: builds and pushes every variant for every PHP version
├── 8.2/ 8.3/ 8.4/ 8.5/           # One directory per PHP version
│   ├── Dockerfile                # Debian base image
│   ├── Dockerfile-alpine         # Alpine base image
│   ├── Dockerfile.node           # Debian + Node.js
│   ├── Dockerfile.node-dev       # Debian + Node.js + Supervisor `npm run dev`
│   ├── Dockerfile.node-alpine    # Alpine + Node.js
│   ├── Makefile                  # Local build/push targets
│   └── configs/
│       ├── php.ini
│       └── supervisor/npm.conf
├── .gitattributes                # Forces LF line endings
├── LICENSE
└── README.md
```

## Building Locally

Each version directory has a Makefile. The targets use `docker buildx` and **push** to Docker Hub, so you need to be logged in with push access (`docker login`).

```bash
cd 8.5
make build              # devasdf/phpswoole:8.5
make build-node         # devasdf/phpswoole:8.5-node
make build-node-dev     # devasdf/phpswoole:8.5-node-dev
make build-alpine       # devasdf/phpswoole:8.5-alpine
make build-node-alpine  # devasdf/phpswoole:8.5-node-alpine
```

The Node variants build `FROM` the published base tag, so build and push the base image first (`build` before `build-node`/`build-node-dev`, and `build-alpine` before `build-node-alpine`).

To build a local image without pushing:

```bash
docker build -t phpswoole:8.5-local -f 8.5/Dockerfile 8.5
```

## Continuous Integration

[`.github/workflows/build.yml`](.github/workflows/build.yml) runs on every push to `main` and can also be started by hand (`workflow_dispatch`). For each PHP version it builds and pushes all five variants. It needs these repository secrets:

- `DOCKER_USERNAME`
- `DOCKER_PASSWORD` (a Docker Hub access token is recommended)

## Contributing

Contributions are welcome:

1. Fork the repository and create a branch.
2. Make your change in **every** affected version directory (`8.2`–`8.5`) so the variants stay consistent.
3. Check that the images build locally, for example with `docker build -f 8.5/Dockerfile 8.5`.
4. Open a pull request that explains the change.

To report bugs or request extensions, please [open an issue](https://github.com/devasdfg/phpswoole/issues).

## License

Released under the [MIT License](LICENSE).

The images include third-party software (PHP, Swoole, Node.js, Debian/Alpine packages and others), which stays under its own licenses.
