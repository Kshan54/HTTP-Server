# CPP Networking Library

A portable, cross-platform networking library written in C++ that provides both TCP and HTTP communication capabilities.

## Features

- **Cross-platform compatibility**: Works on Windows, Linux, and macOS
- **TCP client/server**: Low-level socket communication with an easy-to-use API
- **HTTP server**: Express.js-like API for building HTTP services
- **Event-driven architecture**: Non-blocking I/O for high performance
- **Thread pool**: Efficient connection handling with controlled concurrency
- **File serving**: Built-in static file serving capability
- **Modular design**: Build only what you need

## Requirements

- C++17 compatible compiler
- CMake 3.20 or higher
- Operating system: Windows, Linux, or macOS

## Building the Library

```bash
# Clone the repository
git clone https://github.com/yourusername/cppnetworking.git
cd cppnetworking

# Create a build directory
mkdir build
cd build

# Configure CMake
cmake ..

# Build all components
cmake --build .

# Or build specific components
cmake --build . --target tcpserver
cmake --build . --target tcpclient
cmake --build . --target httpserver
```

## Usage Examples

### TCP Server

```cpp
#include "tcpserver.h"
#include <iostream>

int main() {
    // Create the TCP server using the factory function
    TCPServer* server = createServer();
    
    if (server) {
        // Initialize the server with a specific port
        server->initialize(8080);
        
        std::cout << "Server initialized." << std::endl;
        
        // Start the server (blocking call)
        server->start();

        // Clean up
        delete server;
    }

    return 0;
}
```

### TCP Client

```cpp
#include <iostream>
#include "tcpclient.h"

int main() {
    // Create the TCP client using the factory function
    TCPClient* client = createClient();

    if (!client->initialize()) {
        return 1;
    }

    // Connect to server
    std::string serverIp = "127.0.0.1";
    unsigned short serverPort = 8080;

    if (!client->start(serverIp, serverPort)) {
        return 1;
    }

    // Send data to server
    std::string message = "Hello, server!";
    if (!client->sendData(message)) {
        return 1;
    }

    // Receive data from server
    std::string response = client->recvData();
    if (!response.empty()) {
        std::cout << "Received from server: " << response << std::endl;
    }

    return 0;
}
```

### HTTP Server

```cpp
#include "httpserver.h"
#include <iostream>

int main() {
    const int PORT = 8000;
    HTTPServer *server = new HTTPServer();

    server->get("/users", [](HTTPRequest &req, HTTPResponse &res) {
            std::cout << "Handling /users route" << std::endl;
            res.setStatus(200, "OK");
            res.addHeader("Content-Type", "application/json");
            res.send("{\"users\": [\"Alice\", \"Bob\", \"Charlie\"]}");
        })
        .get("/index.html", [](HTTPRequest &req, HTTPResponse &res) {
            res.sendFile("./public/index.html");
        })
        .post("/adduser", [](HTTPRequest &req, HTTPResponse &res) {
            // Process the request body
            std::string body = req.getBody();
            // Add user logic here...
            
            res.send("User added successfully");
        })
        .listen(PORT, []() {
            std::cout << "HTTP server started on port 8000" << std::endl;
        });

    // The listen call is blocking

    delete server;
    return 0;
}
```

## Architecture

The library follows a layered architecture:

```
+-------------------+
|     HTTP Layer    |
+-------------------+
|     TCP Layer     |
+-------------------+
| Platform-Specific |
|  Implementations  |
+-------------------+
```

See the [High-Level Design Document](HLD.md) for more details about the architecture.

## API Reference

### TCP Server API

- `TCPServer* createServer()`: Factory function to create a platform-specific server instance
- `bool initialize(int port, const std::string &ip_address = "0.0.0.0")`: Initialize the server
- `void start()`: Start the server (blocking call)
- `void setRecvCallback(std::function<void(ClientState&)> callback)`: Set callback for data reception
- `void setSendCallback(std::function<void(int)> callback)`: Set callback for data transmission
- `void setAcceptCallback(std::function<void(int)> callback)`: Set callback for new connections
- `void setErrorCallback(std::function<void(int)> callback)`: Set callback for error handling
- `void setListenCallback(std::function<void(int)> callback)`: Set callback for server start

### TCP Client API

- `TCPClient* createClient()`: Factory function to create a platform-specific client instance
- `bool initialize()`: Initialize the client
- `bool start(const std::string &ip, unsigned short port)`: Connect to server
- `bool sendData(const std::string &data, int length = -1, const std::string &delimiter = "\r\n\r\n")`: Send data
- `std::string recvData()`: Receive data from server
- `void setTimeout(int seconds)`: Set socket timeout
- `bool setSocketNonBlocking()`: Set socket to non-blocking mode

### HTTP Server API

- `HTTPServer& get(const std::string& path, std::function<void(HTTPRequest&, HTTPResponse&)> handler)`: Add GET route
- `HTTPServer& post(const std::string& path, std::function<void(HTTPRequest&, HTTPResponse&)> handler)`: Add POST route
- `void listen(int port, std::function<void()> onStart)`: Start the HTTP server

### HTTP Request API

- `std::string getMethod() const`: Get the HTTP method (GET, POST, etc.)
- `std::string getPath() const`: Get the request path
- `std::string getHeader(const std::string& key) const`: Get a specific header value
- `std::string getBody() const`: Get the request body

### HTTP Response API

- `void addHeader(const std::string& key, const std::string& value)`: Add a response header
- `void setStatus(int statusCode, const std::string& statusMessage)`: Set the response status
- `void send(const std::string& body)`: Send a response with body
- `void send()`: Send a response without body
- `void sendFile(const std::string& filePath)`: Send a file as response

## License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request
