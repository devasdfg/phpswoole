# PHPSwoole: PHP + Swoole + MongoDB

Docker images for high-performance PHP applications, especially **Laravel Octane**, running on [Swoole](https://github.com/swoole/swoole-src). Every image comes with the **MongoDB extension**, **Composer** and a set of common extensions. Variants add **Node.js** for building frontend assets or for local development.

Images are built directly on the **official [`php`](https://hub.docker.com/_/php) images** and rebuilt as soon as the PHP team publishes a release, so security fixes reach these images without waiting on any intermediate image.

## Tags

| PHP | Debian (`linux/amd64`, `linux/arm64`) | Alpine (`linux/amd64`) |
|-----|---------------------------------------|------------------------|
| 8.5 | `8.5`, `8.5-node`, `8.5-node-dev` | `8.5-alpine`, `8.5-node-alpine` |
| 8.4 | `8.4`, `8.4-node`, `8.4-node-dev` | `8.4-alpine`, `8.4-node-alpine` |
| 8.3 | `8.3`, `8.3-node`, `8.3-node-dev` | `8.3-alpine`, `8.3-node-alpine` |
| 8.2 | `8.2`, `8.2-node`, `8.2-node-dev` | `8.2-alpine`, `8.2-node-alpine` |

### Variants

| Tag suffix | Description |
|------------|-------------|
| _(none)_ | PHP + Swoole + MongoDB and the extensions below. For staging and production. |
| `-node` | Same as above, plus **Node.js 22**. For building frontend assets in your image. |
| `-node-dev` | Same as above, plus **Node.js 24** and a Supervisor program that runs `npm run dev` in `/app`. **For local development only.** |
| `-alpine` | Smaller Alpine-based image with the same extensions (no Supervisor). |
| `-node-alpine` | Alpine image plus Node.js and npm from the Alpine repositories. |

## What's Included

- **PHP CLI** from the official `php:X.Y-cli` image (Debian 13 "trixie" or Alpine)
- **Swoole 6.2** with coroutine support for curl, PostgreSQL, SQLite, ODBC, FTP and SSH2, plus OpenSSL, HTTP/2, brotli and zstd
- **MongoDB** extension (latest stable from PECL), with TLS, SRV and zlib/zstd compression
- **Redis** extension, with lzf and zstd compression
- **Other extensions:** GD (FreeType, JPEG, PNG, WebP), Intl, Zip, Gettext, BCMath, PCNTL, Exif, Sockets, PDO MySQL, OPcache, Sodium
- **Composer 2**
- **Supervisor** (Debian images)
- A production-tuned `php.ini`
- Working directory: `/app`

### PHP Configuration

| Setting | Value |
|---------|-------|
| `memory_limit` | `1024M` |
| `max_execution_time` | `0` (unlimited; long-running Swoole workers) |
| `max_input_vars` | `10000` |
| `post_max_size` / `upload_max_filesize` | `300M` |
| `expose_php` | `Off` |
| `display_errors` | `Off` (errors are logged) |

To override settings, add your own `.ini` file:

```dockerfile
COPY my-overrides.ini /usr/local/etc/php/conf.d/zz-overrides.ini
```

## Usage

### Pull the image

```bash
docker pull devasdf/phpswoole:8.4
```

### Run a one-off command

```bash
docker run --rm devasdf/phpswoole:8.4 php -m
```

### Build your application image

```dockerfile
FROM devasdf/phpswoole:8.4-node

COPY . /app
RUN composer install --no-dev --optimize-autoloader \
    && npm ci \
    && npm run build

CMD ["php", "artisan", "octane:start", "--server=swoole", "--host=0.0.0.0", "--port=8000"]
```

### Docker Compose (Laravel Octane)

```yaml
services:
  app:
    image: devasdf/phpswoole:8.4
    ports:
      - "8000:8000"
    volumes:
      - ./:/app
    command: php artisan octane:start --server=swoole --host=0.0.0.0 --port=8000 --workers=2 --max-requests=200
```

### Local development with `-node-dev`

The `-node-dev` image runs `npm run dev` from `/app` under Supervisor and sends its output to the container logs. This works well for a hot-reloading dev server such as Vite next to your Octane app. The Supervisor config is at `/etc/supervisor/conf.d/npm.conf`.

```yaml
services:
  app:
    image: devasdf/phpswoole:8.4-node-dev
    environment:
      DISABLE_DEFAULT_SERVER: "1"   # don't start the sample Swoole server on port 9501
    ports:
      - "8000:8000"   # Octane (start it with: docker compose exec app php artisan octane:start ...)
      - "5173:5173"   # Vite dev server
    volumes:
      - ./:/app
```

### Container start-up (Debian images)

When the container starts **without a command**, Supervisor runs every program configured in `/etc/supervisor/conf.d/`. By default this includes a sample Swoole "Hello, World!" server on port `9501` (`/var/www/server.php`). Set `DISABLE_DEFAULT_SERVER=1` to turn it off, or add your own Supervisor programs.

When the container starts **with a command** (for example `php artisan ...`), only that command runs, and its exit code is passed through.

## Source Code

The Dockerfiles, configuration and build scripts are on [GitHub](https://github.com/devasdfg/phpswoole). The start-up scripts and Supervisor layout of the Debian images come from the official [swoole/docker-swoole](https://github.com/swoole/docker-swoole) project (Apache License 2.0).

- Swoole documentation: <https://wiki.swoole.com>
- Laravel Octane documentation: <https://laravel.com/docs/octane>

## Contributing

Issues and pull requests are welcome on [GitHub](https://github.com/devasdfg/phpswoole/issues).
