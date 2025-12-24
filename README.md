## WebServ

- A lightweight HTTP/1.1 web server implementation in C++98, featuring non-blocking I/O with kqueue, CGI support, and comprehensive HTTP functionality. Built as part of the 42 school curriculum.

## Table of Contents

- [Features](Features)
- [Architecture](Architecture)
- [Installation](Installation)
- [Configuration](Configuration-Directives)
- [Usage](Usage)
- [CGI Support](CGI-Support)
- [HTTP Methods](HTTP-Methods)
- [Project Structure](Project-Structure)
- [Acknowledgments](Acknowledgments)

## Features

Core HTTP Functionality

- HTTP/1.1 Protocol - Full compliance with HTTP/1.1 specifications
- Multiple Virtual Servers - Support for multiple servers on different ports
- Keep-Alive Connections - Persistent connections for improved performance
- Non-Blocking I/O - Asynchronous event-driven architecture using kqueue
- Chunked Transfer Encoding - Support for streaming large payloads
- File Uploads - Multipart form data handling with boundary parsing
- Auto-indexing - Dynamic directory listing generation
- Custom Error Pages - Configurable error responses with custom HTML pages

HTTP methods

- GET - Serve static files, directories, and CGI responses
- POST - Handle file uploads and CGI form submissions
- DELETE - Remove files from the server

Advanced Features

- CGI Execution - Support for Python, PHP, and Perl CGI scripts
- URL Redirections - Configurable HTTP redirects (301, 302, 307, 308)
- Request Body Limits - Configurable maximum body size per location
- Path Normalization - Automatic handling of relative paths and double slashes
- MIME Type Detection - Automatic content-type detection based on file extensions
- Cookie Support - Full cookie handling in CGI scripts
- Timeout Management - 30-second timeouts for connections and CGI execution

## Architecture

Event-Driven Model
The server uses kqueue (BSD's kernel event notification mechanism) to handle multiple concurrent connections efficiently:
```
┌─────────────────────────────────────────┐
│           Main Event Loop (kqueue)      │
├─────────────────────────────────────────┤
│  • EVFILT_READ   - Client data ready    │
│  • EVFILT_WRITE  - Ready to send data   │
│  • EVFILT_PROC   - CGI process exit     │
│  • EVFILT_TIMER  - Connection timeouts  │
└─────────────────────────────────────────┘
            ↓           ↓           ↓
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ Request  │  │ Response │  │   CGI    │
    │ Parser   │  │ Builder  │  │ Handler  │
    └──────────┘  └──────────┘  └──────────┘
Request Processing Pipeline
Client Request → Parse Headers → Validate → Route → Execute → Build Response → Send
                      ↓              ↓         ↓        ↓          ↓
                  Header Map    Syntax    Location  GET/POST/  Generate
                  Query Params   Check    Matching  DELETE/CGI  Headers+Body
```

## Installation

Prerequisites

- C++ Compiler - Supporting C++98 standard (g++, clang++)
- Make - Build automation tool
- Kqueue - Available on macOS and BSD systems

Compile the project
```sh
- git clone <repository-url>
- cd webserv
- make
```

# The executable 'webserv' will be created
```sh
- make clean   # Remove object files
- make fclean  # Remove object files and executable
- make re      # Rebuild from scratch
```
## Configuration

Configuration File Format
The server uses a Nginx-inspired configuration syntax:
```
nginxserver {
    listen 8080;
    listen 8081;
    
    root ./www;
    index index.html;
    autoindex on;
    client_max_body_size 10M;
    
    error_page 404 /errors/404.html;
    error_page 500 /errors/500.html;
    
    location / {
        allow_methods GET POST DELETE;
        root ./www;
        index index.html;
        autoindex on;
        upload_store /tmp/uploads;
    }
    
    location /cgi-bin {
        allow_methods GET POST;
        allow_cgi py php cgi;
        root ./cgi-bin;
        upload_store /tmp/cgi-uploads;
    }
    
    location /redirect {
        return 301 https://example.com;
    }
}
```

## Usage

Starting the Server:
```sh 
- Use default configuration (./Default/default.conf)
./webserv Default/default.conf

```
Use custom configuration file

```sh
./webserv custom.conf
```

# The server will display startup information:
```
[+] Server listening on port 8080
[+] Server listening on port 8081
```
Making Requests
```sh
# GET request
curl http://localhost:8080/

# GET with specific file
curl http://localhost:8080/index.html

# POST file upload
curl -X POST -F "file=@myfile.txt" http://localhost:8080/upload

# DELETE file
curl -X DELETE http://localhost:8080/uploads/myfile.txt

# CGI script execution
curl http://localhost:8080/cgi-bin/script.py?name=value

```
## CGI Support

### Supported Languages
- **Python** (.py) - Via `/usr/bin/python3`
- **PHP** (.php) - Via `/usr/bin/php`
- **Perl** (.cgi) - Direct execution

### CGI Environment Variables
The server provides standard CGI environment variables:
```
REQUEST_METHOD      - HTTP method (GET, POST, DELETE)
SCRIPT_NAME         - Full path to script
SCRIPT_FILENAME     - Script file name
QUERY_STRING        - URL query parameters
SERVER_PROTOCOL     - HTTP/1.1
GATEWAY_INTERFACE   - CGI/1.1
SERVER_SOFTWARE     - webSERV/1.0
PATH_INFO           - Additional path after script
CONTENT_TYPE        - Request content type
CONTENT_LENGTH      - Request body length
SERVER_NAME         - Server hostname
SERVER_PORT         - Server port number
HTTP_COOKIE         - Cookie header
REDIRECT_STATUS     - 200
```

### CGI Timeout
- CGI scripts have a **10-second** execution timeout
- After timeout, a `504 Gateway Timeout` error is returned
- The process is forcefully terminated with SIGKILL

## HTTP Methods

### GET Method
Retrieves resources from the server:
- Static files (HTML, CSS, JS, images, videos)
- Directory listings (if autoindex enabled)
- CGI script outputs

**Features:**
- Content-Type detection based on file extension
- Partial content support for videos
- Automatic index file serving
- Path traversal protection

### POST Method
Submits data to the server:
- **Multipart form uploads** - File uploads with boundary parsing
- **Binary/raw data** - Direct binary uploads
- **Chunked uploads** - Streaming large files
- **CGI form submission** - Form data to CGI scripts

**Upload Formats:**
1. `multipart/form-data` - For file uploads with metadata
2. `application/octet-stream` - Raw binary data
3. `Transfer-Encoding: chunked` - Streaming uploads

### DELETE Method
Removes files from the server:
- Validates file existence and permissions
- Returns success/error response
- Prevents directory deletion

## Project Structure
```
webserv/
├── Makefile                     # Build configuration
├── main.cpp                     # Entry point
├── allincludes.hpp              # Common headers
├── FunctionTools.cpp            # Utility functions
│
├── AllServer/
│   ├── HttpServer.hpp           # Main server class
│   └── HttpServer.cpp           # Event loop & connection handling
│
├── Request/
│   ├── Request.hpp              # Request parser interface
│   ├── Request.cpp              # Header & URI parsing
│   ├── Post.hpp                 # POST handler interface
│   └── Post.cpp                 # Multipart & chunked uploads
│
├── Response/
│   ├── responseHeader.hpp       # Response builder interface
│   ├── responseMain.cpp         # Core response logic
│   ├── get.cpp                  # GET method handler
│   ├── postResponse.cpp         # POST response builder
│   ├── delete.cpp               # DELETE method handler
│   ├── sendResponse.cpp         # Data transmission
│   └── errorPages/              # Custom error HTML pages
│       ├── 400.html
│       ├── 403.html
│       ├── 404.html
│       ├── 405.html
│       ├── 413.html
│       ├── 500.html
│       └── 504.html
│
├── cgi/
│   ├── cgiHeader.hpp            # CGI handler interface
│   ├── cgiMain.cpp              # CGI execution logic
│   ├── cgiExecPars.cpp          # Environment setup & parsing
│   ├── cgiResponse.cpp          # CGI output processing
│   ├── cgiListDir.cpp           # Directory listing for CGI paths
│   └── cgiScripts/              # Example CGI scripts
│       ├── login.py             # Login/session example
│       ├── showenv.py           # Environment display
│       └── redirection.py       # Redirect example
│
├── pars_config/
│   ├── config.hpp               # Configuration parser interface
│   └── config.cpp               # Nginx-style config parsing
│
└── Default/
    └── default.conf             # Default configuration file
```

**Timeout Handling:**
- **Connection timeout**: 30 seconds
- **CGI timeout**: 10 seconds
- Timer reset on each read/write operation

### Security Features
- **Path Traversal Protection** - Rejects `..` in paths
- **Request Size Limits** - Configurable per-location
- **Method Whitelisting** - Explicit allow_methods required
- **File Permission Checks** - Validates read/write/execute
- **Input Validation** - Strict RFC compliance
- **CGI Isolation** - Separate process with environment limits


## Acknowledgments

- RFC 7230-7235 - HTTP/1.1 specification
- CGI/1.1 Specification - Common Gateway Interface
- Nginx - Configuration syntax inspiration