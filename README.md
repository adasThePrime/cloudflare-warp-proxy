# Cloudflare WARP Proxy

A Docker-based proxy service that routes traffic through Cloudflare WARP, providing network privacy and performance. Supports multiple proxy protocols including SOCKS4, SOCKS5, HTTP, and HTTP/2 with optional authentication and TLS encryption.

>[!IMPORTANT]
> - This project is not affiliated with Cloudflare. Cloudflare WARP and related trademarks are property of Cloudflare, Inc.
> - Ensure compliance with Cloudflare's terms of service when using WARP.

## Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
  - [Using Docker](#using-docker)
  - [Using Docker Compose](#using-docker-compose)
  - [Available Image Tags](#available-image-tags)
- [Configuration](#configuration)
  - [Proxy Types and Ports](#proxy-types-and-ports)
  - [Customizing Base Ports](#customizing-base-ports)
  - [Authentication](#authentication)
  - [TLS Configuration](#tls-configuration)
  - [Logging](#logging)
- [Advanced Usage](#advanced-usage)
  - [DNS Configuration](#dns-configuration)
  - [Persistent Data](#persistent-data)
  - [Multiple Proxy Instances](#multiple-proxy-instances)
  - [Building from Source](#building-from-source)
- [License](#license)

## Features

- **Multiple Proxy Protocols**: SOCKS4, SOCKS5, HTTP, and HTTP/2
- **Flexible Authentication**: Optional username/password authentication for supported protocols
- **TLS Encryption**: Secure your proxy connections with TLS
- **Cloudflare DNS**: Built-in DNS over HTTPS, TLS, and UDP fallback
- **Easy Deployment**: Simple Docker and Docker Compose setup
- **Automatic Updates**: Pre-built images updated with latest WARP client versions

## Quick Start

### Using Docker

Run the container with a single SOCKS5 proxy on port 1180:

```bash
docker run -d \
    --name cloudflare-warp-proxy \
    -e SOCKS5_PORTS=1 \
    -p 1180:1180 \
    ghcr.io/adasThePrime/cloudflare-warp-proxy:latest
```

Test the proxy:

```bash
curl -x socks5://localhost:1180 https://reqtrace.qzz.io
```

### Using Docker Compose

1. Clone the repository:

```bash
git clone https://github.com/adasThePrime/cloudflare-warp-proxy.git
cd cloudflare-warp-proxy
```

2. Start the service:

```bash
docker-compose up -d
```

3. Check the logs:

```bash
docker-compose logs -f
```

### Available Image Tags

- `latest`: Most recent build with the newest WARP client version
- `<warp_version>`: Specific WARP client version (e.g., `2025.8.779.0`)
- `build-<hash>`: Specific build identified by commit hash

Images are automatically built and published when new WARP client versions are released.

## Configuration

### Proxy Types and Ports

Enable proxy types by setting the number of ports for each. Set to `0` or omit to disable a proxy type.

| Environment Variable | Description | Default Base Port |
|---------------------|-------------|-------------------|
| `SOCKS4_PORTS` | Number of SOCKS4 proxy ports | 1080 |
| `SOCKS5_PORTS` | Number of SOCKS5 proxy ports | 1180 |
| `HTTP_PORTS` | Number of HTTP proxy ports | 1280 |
| `SOCKS4_TLS_PORTS` | Number of TLS-enabled SOCKS4 ports | 1380 |
| `SOCKS5_TLS_PORTS` | Number of TLS-enabled SOCKS5 ports | 1480 |
| `HTTP_TLS_PORTS` | Number of TLS-enabled HTTP ports | 1580 |
| `HTTP2_TLS_PORTS` | Number of TLS-enabled HTTP/2 ports | 1680 |

**Important Notes:**
- At least one proxy type must be enabled
- HTTP/2 requires TLS to be enabled
- SOCKS4 does not support authentication

**Port Allocation Example:**

If you set `SOCKS5_PORTS=5` with `SOCKS5_BASE_PORT=1180`, the proxy will listen on ports 1180, 1181, 1182, 1183, and 1184.

### Customizing Base Ports

Override the default starting port for each proxy type:

| Environment Variable | Default |
|---------------------|---------|
| `SOCKS4_BASE_PORT` | 1080 |
| `SOCKS5_BASE_PORT` | 1180 |
| `HTTP_BASE_PORT` | 1280 |
| `SOCKS4_TLS_BASE_PORT` | 1380 |
| `SOCKS5_TLS_BASE_PORT` | 1480 |
| `HTTP_TLS_BASE_PORT` | 1580 |
| `HTTP2_TLS_BASE_PORT` | 1680 |

### Authentication

Add password protection to your proxies by setting both username and password environment variables. Authentication is supported for all proxy types except SOCKS4.

| Proxy Type | Username Variable | Password Variable |
|-----------|------------------|-------------------|
| SOCKS5 | `SOCKS5_USER` | `SOCKS5_PASS` |
| HTTP | `HTTP_USER` | `HTTP_PASS` |
| SOCKS5 TLS | `SOCKS5_TLS_USER` | `SOCKS5_TLS_PASS` |
| HTTP TLS | `HTTP_TLS_USER` | `HTTP_TLS_PASS` |
| HTTP/2 TLS | `HTTP2_TLS_USER` | `HTTP2_TLS_PASS` |

**Example with Authentication:**

```bash
docker run -d \
    --name cloudflare-warp-proxy \
    -e SOCKS5_PORTS=1 \
    -e SOCKS5_USER=myuser \
    -e SOCKS5_PASS=mypassword \
    -p 1180:1180 \
    ghcr.io/adasThePrime/cloudflare-warp-proxy:latest
```

Test with authentication:

```bash
curl -x socks5://myuser:mypassword@localhost:1180 https://reqtrace.qzz.io
```

### TLS Configuration

TLS-enabled proxy types require a certificate and private key. You can obtain a free TLS certificate from [here](https://cfwapca.qzz.io) or use your own existing certificate.

There are two methods to provide these files:

#### Method 1: Build-Time Certificates

1. Create a `certs` directory in the project root
2. Add your certificate files:
   - `server.crt` - TLS certificate
   - `server.key` - TLS private key (optionally encrypted)
3. Build the Docker image

#### Method 2: Run-Time Certificates

Mount your certificate directory when running the container:

```bash
docker run -d \
    --name cloudflare-warp-proxy \
    -e SOCKS5_TLS_PORTS=1 \
    -e TLS_KEY_PASS=your_key_password \
    -v /path/to/your/certs:/user-certs:ro \
    -p 1480:1480 \
    ghcr.io/adasThePrime/cloudflare-warp-proxy:latest
```

The mounted directory must contain:
- `server.crt` - TLS certificate
- `server.key` - TLS private key (optionally encrypted)

**TLS Environment Variables:**

- `TLS_KEY_PASS`: Password for encrypted private key (optional)
- `FORCE_CERT_REPLACE`: Set to `true` to force certificate replacement on startup (default: `false`)

### Logging

Control the verbosity of proxy server logs:

```bash
-e GOST_LOGGER_LEVEL=debug
```

**Available Log Levels:**
- `trace` - Most verbose, includes all debug information
- `debug` - Detailed debugging information
- `info` - General informational messages (default)
- `warn` - Warning messages only
- `error` - Error messages only
- `fatal` - Critical errors only

## Advanced Usage

### DNS Configuration

The proxy includes Cloudflare DNS with automatic fallback:

1. DNS over HTTPS (primary)
2. DNS over TLS (fallback)
3. Standard UDP DNS (final fallback)

This ensures reliable DNS resolution even if one method fails.

### Persistent Data

The container uses Docker volumes for persistent storage:

- `warp-data`: Stores WARP client registration and configuration
- `certs`: Stores TLS certificate files

### Multiple Proxy Instances

Run multiple proxy types simultaneously:

```bash
docker run -d \
    --name cloudflare-warp-proxy \
    -e SOCKS5_PORTS=3 \
    -e HTTP_PORTS=2 \
    -e SOCKS5_USER=user1 \
    -e SOCKS5_PASS=pass1 \
    -e HTTP_USER=user2 \
    -e HTTP_PASS=pass2 \
    -p 1180-1182:1180-1182 \
    -p 1280-1281:1280-1281 \
    ghcr.io/adasThePrime/cloudflare-warp-proxy:latest
```

This creates:
- 3 SOCKS5 proxies on ports 1180-1182 (with authentication)
- 2 HTTP proxies on ports 1280-1281 (with authentication)

### Building from Source

Clone and build the image locally:

```bash
git clone https://github.com/adasThePrime/cloudflare-warp-proxy.git
cd cloudflare-warp-proxy
docker build -t cloudflare-warp-proxy:custom .
```

Run your custom build:

```bash
docker run -d \
    --name cloudflare-warp-proxy \
    -e SOCKS5_PORTS=1 \
    -p 1180:1180 \
    cloudflare-warp-proxy:custom
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.