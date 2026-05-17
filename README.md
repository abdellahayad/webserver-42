
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

Development workflow (how to work on this project)
-------------------------------------------------
Follow these steps when you develop, debug, or contribute code.

1. Setup (prerequisites)

```bash
# Install standard build tools (example for Debian/Ubuntu)
sudo apt update && sudo apt install -y build-essential g++ make gdb valgrind
```

2. Build

```bash
make
```

3. Run locally

```bash
./webserv [config/exemple.conf]
```

4. Quick manual tests

```bash
curl -i http://localhost:8080/
curl -i -X POST -d 'key=val' http://localhost:8080/endpoint
```

5. Debugging and memory checks

- Use `gdb` to step through `main` and connection handling.
- Use `valgrind --leak-check=full ./webserv` to check for leaks.

6. Adding a feature or fixing a bug

- Create a feature branch: `git checkout -b feat/<short-descr>`
- Add tests or manual reproduction steps in a new `test/` file if applicable.
- Keep commits focused and document behavior in the commit message.
- Update `README.md` or add docs for new configuration options.
- Open a pull request describing the change and how to test it.

7. Code style & review notes

- Follow the existing project style (clear names, modular functions).
- Keep functions small and single-responsibility.
- Add brief header comments for complex modules (see header template below).

Recommended professional file header (add to top of modified source files)

```
/*
 * Project: webserv
 * File: <filename>
 * Description: <one-line purpose of this file>
 * Author: <Your Name> (<email>)
 * Created: YYYY-MM-DD
 * License: See LICENSE in project root
 */
```

Contributing
------------
Contributions are welcome. Keep changes focused, add tests or manual steps to reproduce behavior, and open a PR with a clear description. For large changes, open an issue first to discuss design.

Authors
-------
- ayad — response handling and CGI
- zel-yama — configuration parser and server core
- mohidbel — request parsing, sessions, cookies

Resources & RFCs
----------------
Authoritative specs and useful references used when implementing or studying this project.

Standards (RFCs)
- HTTP/1.1 message syntax and semantics: [RFC 7230](https://datatracker.ietf.org/doc/html/rfc7230), [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) (methods, status codes), [RFC 7234](https://datatracker.ietf.org/doc/html/rfc7234) (caching), [RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235) (authentication)
- CGI: [RFC 3875](https://datatracker.ietf.org/doc/html/rfc3875)
- Multipart/form-data (file uploads): [RFC 7578](https://datatracker.ietf.org/doc/html/rfc7578)
- MIME / media types: [RFC 2045](https://datatracker.ietf.org/doc/html/rfc2045) and IANA media types registry (text, image, application) — https://www.iana.org/assignments/media-types/
- URIs: [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986)

Linux / system programming
- `epoll` man page (overview): https://man7.org/linux/man-pages/man7/epoll.7.html
- `epoll(2)` and related syscalls: https://man7.org/linux/man-pages/man2/epoll_create.2.html (see `epoll_create1`, `epoll_ctl`, `epoll_wait`)
- `socket(2)` man page: https://man7.org/linux/man-pages/man2/socket.2.html
- Non-blocking I/O and I/O multiplexing tutorials (practical guides):
	- "Non-Blocking Sockets and I/O Multiplexing with epoll in C" — tutorial article (searchable)
	- "The C10k Problem" — overview of scaling network servers: http://www.kegel.com/c10k.html

Developer-friendly docs and tutorials
- MDN Web Docs — HTTP overview: https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
- MDN — HTTP methods: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- MDN — HTTP status codes: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

Additional reading
- CGI examples and best practices: search for RFC 3875 examples and server-specific CGI guides (Apache, Nginx)
- Multipart/form-data and form handling: see RFC 7578 and practical guides on handling uploads in C/C++

