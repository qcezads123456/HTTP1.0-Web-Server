# HTTP/1.0 Web Server

A simple HTTP/1.0 web server written in C as a self-practice project for learning socket programming, multithreading, and basic web server implementation.

## Features

- HTTP/1.0 request handling
- Static file serving
- Thread pool for concurrent connections
- Request timeout handling
- Path whitelist
- Graceful shutdown with `Ctrl+C`
- Optional Nginx reverse proxy configuration

## Quick Start

### Requirements

Linux / WSL with:

```bash
gcc
make
```

Clone the repository:

```bash
git clone https://github.com/qcezads123456/HTTP1.0-Web-Server.git
cd HTTP1.0-Web-Server
```

Build:

```bash
make
```

Run:

```bash
make run
```

or:

```bash
./build/server
```

The server listens on port `8080`:

```text
http://localhost:8080
```

Static files are served from the `www/` directory.

For example:

```text
www/
└── index.html
```

## Nginx Reverse Proxy

The server can run independently on port `8080`.

An example Nginx configuration is provided in:

```text
config/config.conf
```

Nginx can be used as a reverse proxy:

```text
Client → Nginx (:80) → HTTP Server (:8080)
```

This allows the web server to be accessed through the standard HTTP port without explicitly specifying `:8080`.

To install the example configuration, copy it into your system's Nginx configuration directory.

For example:

```bash
sudo cp config/config.conf /etc/nginx/conf.d/http-server.conf
```

Test the configuration:

```bash
sudo nginx -t
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

> Installing or modifying system Nginx configuration requires root permission.  
> The exact Nginx configuration path may vary depending on the Linux distribution.

## Make Commands

```bash
make          # Build the server
make run      # Build and run
make clean    # Remove build files
make rebuild  # Clean and rebuild
```

## Project Structure

```text
HTTP1.0-Web-Server
├── Makefile
├── README.md
├── build/              # Stores compiled executable files
├── config/             # Stores nginx config files
│   └── config.conf
├── include/            # Header files
│   ├── main.h
│   ├── parser.h
│   ├── socket.h
│   ├── thread_pool.h
│   └── whitelist.h
├── src/                # Source files
│   ├── main.c
│   ├── parser.c
│   ├── socket.c
│   ├── thread_pool.c
│   └── whitelist.c
├── whitelist/          # Stores allowed paths to prevent path injection
│   └── whitelist.txt
└── www/                # Input files for parsing
```

## Performance Benchmark

Performance testing was conducted using **ApacheBench (ab)** against the HTTP/1.0 server directly on `localhost:8080`, without the Nginx reverse proxy.

### Test Configuration

| Parameter | Value |
|---|---|
| Benchmark Tool | ApacheBench 2.3 |
| Environment | WSL (Linux) |
| Target | `http://localhost:8080/` |
| Total Requests | 100,000 per test |
| Concurrency Levels | 1, 10, 50, 100, 500, 1000 |
| Response Body | 121 bytes |
| HTTP Connection | Non-persistent (`Connection: close`) |

### Benchmark Results

| Concurrency | Requests/sec | Avg Latency | P99 Latency | Failed Requests |
|---:|---:|---:|---:|---:|
| 1 | 2,117.67 | 0.47 ms | 1 ms | 0 |
| 10 | 5,962.76 | 1.68 ms | 4 ms | 0 |
| 50 | 6,539.11 | 7.65 ms | 14 ms | 0 |
| 100 | 6,918.88 | 14.45 ms | 23 ms | 0 |
| 500 | 7,248.03 | 68.98 ms | 93 ms | 0 |
| 1000 | **7,369.60** | 135.69 ms | 170 ms | 0 |

### Observations

- **Peak measured throughput:** Approximately 7,370 requests per second at 1,000 concurrent connections.
- **Reliability:** ApacheBench reported zero failed requests across all six tests (600,000 requests in total).
- **Throughput scaling:** Throughput increases with concurrency but begins to level off at higher concurrency levels.
- **Latency trade-off:** Higher concurrency increases request latency while providing diminishing throughput improvements.

### Reproduce the Benchmark

Example command:

`ab -n 100000 -c 1000 http://localhost:8080/`

These results represent local loopback performance under a lightweight static-response workload. Actual performance may vary depending on hardware, operating system, logging overhead, and network conditions.

### Future Improvements

- Dynamic thread pool scaling
- Improved thread synchronization and graceful shutdown
- Further performance profiling and optimization