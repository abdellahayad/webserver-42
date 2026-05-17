*webserv* — a compact educational HTTP server written in C++.

Overview
--------
webserv is an instructional web server implemented for the 42 curriculum. It focuses on clarity and correctness, demonstrating how a production-style server handles socket I/O, request parsing, routing, static files, CGI execution, and concurrent clients using epoll.

Core features
-------------
- Event-driven I/O using epoll (non-blocking sockets)
- HTTP/1.1 request parsing and response generation
- GET, POST, DELETE handling and configurable limits
- CGI support for running external scripts/programs
- Session tracking via cookies and a simple session manager
- Static file serving and configurable error pages

Architecture summary
--------------------
The codebase is organized to separate concerns and make it easy to follow:

| Path / Folder | Responsibility |
|---|---|
| `server/` | Server bootstrap, `main.cpp`, and connection lifecycle |
| `server/serverSetup/` | Event loop, read/write helpers, and connection state |
| `request/` | Request parsing, validation, multipart handling, sessions |
| `Response/` | Response assembly, headers, static file serving, CGI glue |
| `config/` | Example configs and parser inputs |
| `tools/` | Small utilities and helpers |
| `site/`, `errors/` | Example static site and error pages used in tests |

Key concepts (concise)
----------------------
- HTTP: a text protocol. Client sends a request line, headers, optional body; server responds with status, headers, and body.
- CGI: server executes an external program, provides request data via environment variables/stdin, and relays stdout back to the client. Simple and compatible; not the most efficient for high throughput.
- epoll: Linux I/O multiplexing API. Use non-blocking sockets and epoll to get notifications for readability/writability and scale to many connections without threads per socket.

HTTP reference tables
---------------------

Methods
| Method | Description |
|---|---|
| GET | Read a resource |
| POST | Send data to the server (forms, files) |
| DELETE | Remove a resource |

Status codes (common)
| Code | Meaning |
|---:|---|
| 200 | OK |
| 201 | Created |
| 301 | Moved Permanently |
| 400 | Bad Request |
| 403 | Forbidden |
| 404 | Not Found |
| 413 | Payload Too Large |
| 500 | Internal Server Error |

CGI flow (high-level)
----------------------
1. Server recognizes a CGI-enabled path.
2. Build environment variables: `REQUEST_METHOD`, `QUERY_STRING`, `CONTENT_LENGTH`, etc.
3. Spawn the process, pipe request body to its stdin.
4. Read the program's stdout (headers + body) and forward to the client.

Build and run
-------------
Build the project using the provided `Makefile`:

```bash
make
```

Run the server (optional: pass a config file):

```bash
./webserv [config/zel-yama.conf]
```

Quick test
---------
Fetch the root index:

```bash
curl -i http://localhost:8080/
```

Configuration
-------------
Example configuration files live in the `config/` folder. Typical server block options include `listen`, `server_name`, `root`, `location`, `allowed_methods`, and `body_max_byte`.

Where to start in the code
--------------------------
- `server/main.cpp`: program entry and server bootstrap
- `server/serverSetup/eventLoop.cpp`: epoll loop and event dispatch
- `request/RequestParser.cpp`: parsing logic for request line, headers, and body
- `Response/cgi.cpp`: CGI invocation and output handling

Contributing
------------
Contributions are welcome. Keep changes focused, add tests or manual steps to reproduce behavior, and open a PR with a clear description. For large changes, open an issue first to discuss design.

Authors
-------
- ayad — response handling and CGI
- zel-yama — configuration parser and server core
- mohidbel — request parsing, sessions, cookies

References
----------
- HTTP: RFC 7230–7235
- CGI: RFC 3875
- epoll: Linux man pages and epoll tutorials

License
-------
See the project root for license information (add a LICENSE file if needed).

If you want a more concise developer README, diagrams (sequence/architecture), or example requests and responses added, tell me which and I will add them.

