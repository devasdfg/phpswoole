# PHP Swoole Base Images

This repository contains production-ready Docker base images for PHP applications using Swoole. The images are built on top of the official `phpswoole/swoole` images with additional extensions and configurations optimized for production use.

## Available Versions

- PHP 8.2 with Swoole 5.1
- PHP 8.3 with Swoole 5.1

## Features

- Based on official PHP Swoole images
- Additional PHP extensions installed:
  - GD (with FreeType and JPEG support)
  - Intl
  - ZIP
  - Gettext
  - BCMath
  - PCNTL
  - Exif
  - MongoDB
- Custom PHP configuration included
- Node.js variant available for frontend builds
- Production-optimized settings

## Usage

### Basic PHP Swoole Image

```dockerfile
FROM your-registry/swoole-base:8.2
# or
FROM your-registry/swoole-base:8.3

# Add your application code
COPY . /app
```

### Node.js Variant (for frontend builds)

```dockerfile
FROM your-registry/swoole-base:8.2-node
# or
FROM your-registry/swoole-base:8.3-node

# Add your application code and build frontend assets
COPY . /app
```

## Building Images

Each version directory contains a Makefile for easy building:

```bash
cd 8.2  # or 8.3
make build
```

## Configuration

- Custom PHP configuration is provided via `php.ini`
- Working directory is set to `/app`
- All necessary system libraries are included for the installed PHP extensions

## License

[Add your license information here]

## Contributing

[Add contribution guidelines here]
