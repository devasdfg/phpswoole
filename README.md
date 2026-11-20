# PHP Swoole Base Images

[![Build and push images](https://github.com/devasdfg/phpswoole/actions/workflows/build.yml/badge.svg)](https://github.com/devasdfg/phpswoole/actions/workflows/build.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Docker base images for PHP applications running on [Swoole](https://github.com/swoole/swoole-src). They build directly on the official [`php`](https://hub.docker.com/_/php) images and add Swoole, MongoDB, Redis and other common PHP extensions, Composer, Supervisor, a production-tuned `php.ini`, and optional Node.js variants for building frontend assets.

Because nothing sits between the official PHP image and these images, a PHP security release can be picked up as soon as the PHP team publishes it, without waiting for an intermediate image to be rebuilt.

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
| _(none)_ | `php:X.Y-cli-trixie` | PHP + Swoole with the extra extensions, Composer and Supervisor |
| `-node` | Debian image above | Adds Node.js 22 (via NodeSource) for frontend builds |
| `-node-dev` | Debian image above | Adds Node.js 24 (via NodeSource) plus a Supervisor program that runs `npm run dev` in `/app` |
| `-alpine` | `php:X.Y-cli-alpine` (8.4); `phpswoole/swoole:6.2-phpX.Y-alpine` (8.2, 8.3, 8.5, not migrated yet) | Smaller Alpine-based image with the same extensions |
| `-node-alpine` | Alpine image above | Adds Node.js and npm from the Alpine repositories |

## Features

- Built on the official `php` CLI images (Debian 13 "trixie" and Alpine)
- **Swoole 6.2.6**, built from source with the same options as the official `phpswoole/swoole` images: coroutine curl, PostgreSQL, SQLite, ODBC, FTP and SSH2, plus OpenSSL, HTTP/2, brotli and zstd
- PHP extensions:
  - MongoDB (latest stable from PECL)
  - Redis 6.3.0 (lzf and zstd compression)
  - GD (FreeType, JPEG, PNG and WebP support)
  - Intl
  - Zip
  - Gettext
  - BCMath
  - PCNTL
  - Exif
  - Sockets
  - PDO MySQL
  - OPcache and Sodium (from the official image)
- Composer 2
- Supervisor, with the start-up behaviour of the `phpswoole/swoole` images (Debian images; see [Container start-up](#container-start-up))
- Custom `php.ini` installed at `/usr/local/etc/php/php.ini`
- Working directory set to `/app`
- Multi-arch Debian images (`linux/amd64`, `linux/arm64`)
- The build fails if any expected extension is missing or PHP prints a start-up warning
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

## Container start-up

The Debian images use the entrypoint of the `phpswoole/swoole` images (`/entrypoint.sh`):

- **Without a command**, Supervisor starts every program in `/etc/supervisor/conf.d/`. By default this includes a sample Swoole "Hello, World!" server on port `9501` (`/var/www/server.php`). Set `DISABLE_DEFAULT_SERVER=1` to turn it off.
- **With a command** (for example `php artisan octane:start ...`), only that command runs, and its exit code is passed through.
- Scripts in `/usr/local/boot/` run on every start. Programs in `/etc/supervisor/available.d/` can be enabled with `enable-supervisord-program.sh <name>`.

The Alpine images have no Supervisor and use the official image's default entrypoint.

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
│   ├── configs/
│   │   ├── php.ini
│   │   └── supervisor/npm.conf
│   ├── rootfs/                   # Entrypoint, boot scripts, Supervisor layout and helper scripts (from docker-swoole)
│   └── rootfs-legacy/            # Legacy helper scripts the phpswoole/swoole 6.2 images still ship
├── .gitattributes                # Forces LF line endings
├── DOCKERHUB.md                  # Overview text for the Docker Hub repository page
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
docker build --pull -t phpswoole:8.5-local -f 8.5/Dockerfile 8.5
```

### Versions and build arguments

The Debian and Alpine Dockerfiles take these build arguments:

| Argument | Default | Purpose |
|----------|---------|---------|
| `PHP_VERSION` | the directory's version | Tag of the official `php` image (`X.Y` follows the latest patch release) |
| `SWOOLE_VERSION` | `6.2.6` | Swoole release to build from source |
| `REDIS_VERSION` | `6.3.0` | Redis extension release |
| `MONGODB_PACKAGE` | `mongodb` | PECL package spec; set e.g. `mongodb-2.5.8` to pin a version |
| `COMPOSER_VERSION` | `2` | Tag of the official `composer` image to copy Composer from |
| `DEBIAN_RELEASE` | `trixie` | Debian release of the official `php` image (Debian images only) |

For example, to try a newer Swoole release:

```bash
docker build --pull --build-arg SWOOLE_VERSION=6.2.7 -t phpswoole:8.4-swoole627 -f 8.4/Dockerfile 8.4
```

## Picking Up a PHP Security Release

When the PHP team publishes a release, the official `php:X.Y-cli` tags are updated within hours. To pick it up, rebuild with `--pull` so Docker fetches the new base instead of using a cached one. The Makefile `build` and `build-alpine` targets and the CI workflow already do this. Rebuild the Node.js variants after their base image has been pushed.

## Releasing Safely

The `X.Y` and `X.Y-node` tags are used in staging and production. Before changing them, publish release candidates and test those first:

1. Build the base locally and push it as `X.Y-rc`, then build `Dockerfile.node` on top of it and push it as `X.Y-node-rc`:

   ```bash
   cd 8.4
   docker buildx build --pull --platform linux/amd64,linux/arm64 -t phpswoole-local:8.4 -f Dockerfile --load .
   docker tag phpswoole-local:8.4 docker.io/devasdf/phpswoole:8.4-rc && docker push docker.io/devasdf/phpswoole:8.4-rc
   docker buildx build --platform linux/amd64,linux/arm64 \
     --build-context docker.io/devasdf/phpswoole:8.4=docker-image://phpswoole-local:8.4 \
     -t docker.io/devasdf/phpswoole:8.4-node-rc -f Dockerfile.node . --push
   ```

2. Point staging at the `-rc` tags and test the application.
3. Note the current digests so you can roll back:

   ```bash
   docker buildx imagetools inspect docker.io/devasdf/phpswoole:8.4 --format '{{json .Manifest.Digest}}'
   ```

4. Promote by re-tagging, not rebuilding, so production gets exactly the image that was tested:

   ```bash
   docker buildx imagetools create -t docker.io/devasdf/phpswoole:8.4 docker.io/devasdf/phpswoole:8.4-rc
   docker buildx imagetools create -t docker.io/devasdf/phpswoole:8.4-node docker.io/devasdf/phpswoole:8.4-node-rc
   ```

5. To roll back, point the tag at the old digest again:

   ```bash
   docker buildx imagetools create -t docker.io/devasdf/phpswoole:8.4 docker.io/devasdf/phpswoole@sha256:<old digest>
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

The images include third-party software (PHP, Swoole, Node.js, Debian/Alpine packages and others), which stays under its own licenses. The files under `rootfs/` and `rootfs-legacy/` come from [swoole/docker-swoole](https://github.com/swoole/docker-swoole) and are licensed under the Apache License 2.0.
