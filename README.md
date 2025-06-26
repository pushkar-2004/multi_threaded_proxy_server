# Open
Open http://localhost:port/https://www.cs.princeton.edu/

# Multi-threaded Proxy Server

This repository contains a high-performance, multi-threaded HTTP proxy server written in C, with both caching and non-caching variants. The server is designed to handle multiple client connections concurrently, supporting up to 400 simultaneous connections, and can efficiently cache web responses to improve performance on repeated requests.

## Features

- **Multi-threaded Design**: Handles multiple clients concurrently using POSIX threads (`pthread`).
- **Semaphore-Based Concurrency Control**: Limits the number of active clients to prevent overload.
- **HTTP/1.0 and HTTP/1.1 Support**: Accepts standard HTTP requests and forwards them to the destination server.
- **Response Caching (optional)**: Caching variant stores responses in memory for fast retrieval on repeated requests (LRU-style management).
- **Error Handling**: Graceful error responses for unsupported methods and protocol errors.
- **Configurable Port**: Run the proxy server on any port specified at startup.
- **Resource Management**: Uses mutexes and semaphores for thread-safe cache and connection management.

## File Structure

```
.
├── proxy_server_with_cache.c      # Proxy server with in-memory caching
├── proxy_server_without_cache.c   # Proxy server without caching
├── proxy_parse.h                  # Request parsing utilities (required for compilation)
├── [other files: Makefile, README, etc.]
```

## Dependencies

- POSIX threads (`pthread`)
- POSIX semaphores (`semaphore.h`)
- Standard C libraries
- Linux sockets API

## Usage

### Compilation

You can compile either the caching or non-caching proxy server. Replace `<source_file>` with either `proxy_server_with_cache.c` or `proxy_server_without_cache.c`.

```sh
gcc -o proxy_server <source_file> -lpthread
```

### Running the Server

Start the proxy server on a specified port (e.g., 8080):

```sh
./proxy_server 8080
```

- The server will bind to all interfaces on the specified port.
- For each incoming connection, a new thread is spawned to handle the client.
- If the number of clients exceeds the maximum (default: 400), new connections are blocked until a slot is free.

### Using the Proxy

Configure your browser or HTTP client to use `localhost:8080` (or the port you specified) as the HTTP proxy.

## How It Works

- **Connection Handling**: The server listens for incoming client connections, accepting each in a loop and assigning it to a thread.
- **Request Parsing**: Each thread parses the HTTP request, checks for supported methods (only `GET` is supported), and forwards the request to the target server.
- **Caching**: (Caching version only) Before forwarding, the server checks if the response is stored in the in-memory cache. If found, it returns the cached response. Otherwise, it fetches the response, stores it in the cache, and sends it to the client.
- **Cache Management**: Uses a linked list for cache elements, protected by mutexes for thread safety. LRU (Least Recently Used) logic is applied for eviction when the cache is full.

## Configuration

- **MAX_CLIENTS**: Maximum number of simultaneous client connections (default: 400).
- **MAX_BYTES**: Maximum allowed size for request/response buffers (default: 4096 bytes).
- **Cache Size**: Defaults to ~200MB total, with a maximum element size of ~10MB (in caching variant).

You can adjust these by modifying the `#define` statements at the top of the source files.

## Limitations

- Only supports the `GET` HTTP method.
- No HTTPS (TLS/SSL) proxying support.
- Cache is in-memory only; it is cleared when the server restarts.
- No authentication or access control.

## License

This project is open-source. See `LICENSE` for details (if provided).

---

**Author:** [pushkar-2004](https://github.com/pushkar-2004)