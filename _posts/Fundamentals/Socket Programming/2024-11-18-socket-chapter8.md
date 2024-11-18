---
title: "Socket Programming : Port"

categories:
    - socket

tags:
    - [Socket Programming, c++, socket, windows, port]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-18
---

# Port
When running a `local server`, it is generally **okay** to use **any available port number**, but there are some important considerations to keep in mind:

## **Port Number Range and Availability**

### **Well-known ports**:
Ports in the range `0-1023` are typically reserved for system or well-known services 
- like **HTTP** on port `80`, **HTTPS** on port `443`, **FTP** on port `21`
- It's a good idea to **avoid** these ports unless you're specifically running one of these services.

### **Registered ports**:
Ports from `1024-49151` are assigned by **IANA** (Internet Assigned Numbers Authority) for specific services, but they are **less restrictive** than the well-known ports.
- You **can use** them, but it’s important to **check** if the port is **already in use** by another application to avoid `conflicts`.

### **Dynamic or private ports**: 
Ports in the range `49152-65535` are considered dynamic or private and are typically **available** for general use.
- These ports are less likely to be in use by default.


## **Port Conflicts**
If you try to run a local server on a port that is **already occupied** by another service, your server will `fail` to start, and you will get an `error` saying the port is in use.
   
### How to Avoid Port Conflicts
- **Check for conflicts**
    * Before using a port, you can check if it's in use using **commands** like:
        + On Linux/macOS: `lsof -i :<port>` or `netstat -an | grep <port>`
        + On Windows: `netstat -ano | findstr :<port>`
    * you can check this in `C++` code as well
        ```c++
        bool isPortInUse(int port) {
            // Try IPv6
            int sockfd = socket(AF_INET6, SOCK_STREAM, 0); 
            if (sockfd < 0) {
                std::cerr << "Socket creation failed: " << strerror(errno) << std::endl;
                throw std::runtime_error("Socket creation failed");
            }

            sockaddr_storage addr;
            memset(&addr, 0, sizeof(addr));
            sockaddr_in6* addr_in6 = reinterpret_cast<sockaddr_in6*>(&addr);
            addr_in6->sin6_family = AF_INET6;
            addr_in6->sin6_port = htons(port);
            addr_in6->sin6_addr = in6addr_any;          // Listen on any IPv6 interface

            // Try to bind the socket to the given port to check whether this port is in use or not
            int result = bind(sockfd, reinterpret_cast<sockaddr*>(&addr), sizeof(addr));
            if (result == 0) {
                // If bind succeeds, this means that the port is free
                // Don't forget to close the socket after testing
                close(sockfd); 
                return false;
            } else {
                // If bind fails and error is EADDRINUSE, the port is in use
                if (errno == EADDRINUSE) {
                    close(sockfd);  // Close the socket
                    return true;
                } else {
                    std::cerr << "Bind failed: " << strerror(errno) << std::endl;
                    close(sockfd);
                    throw std::runtime_error("Bind failed");
                }
            }
            return false;
        }
        ```
- **Pick a random, unused port**
    * You can choose any port number from the **dynamic range**, ensuring that it's not already occupied by another process.

### **Standard Practices**
Some ports are **frequently used** for `local development`, such as:
- `3000`, `5000`, `8080`, `8000`, `9000`, etc.
    * These are **common choices** for web servers, APIs, or other development tools, and they are widely accepted in the development community.
- **Avoiding conflicts**
    * If you're working on a project with multiple developers or multiple services running locally
        + e.g., a backend, frontend, database, etc.
    * it's useful to ***standardize*** or **coordinate** port usage to avoid `conflicts`.

### **Special Considerations for Certain Frameworks or Tools**
Some `web frameworks` or tools may have their own **default ports**.
- **Node.js** (Express.js)
    * often defaults to port `3000`.
- **Ruby on Rails**
    * uses port `3000` by default.
- **Django**
    * uses port `8000` by default for local development.
- **React** development server
    * typically uses `3000`.

In such cases, it’s a good idea to either:
- **Stick to the defaults** if you're just running one instance.
- **Manually configure the port for each service** if you need to run multiple instances on the same machine.


## **Firewall and Security**
Local servers running on a **port** that’s **not protected** by a `firewall` may be **vulnerable** to external access if exposed.
- For a purely local server (e.g., for development), this is usually not a concern
- but if you're opening ports to **external traffic** (e.g., in production)
    * it’s important to consider `firewall rules` and `security implications`.
   
### **Security Implications**
Although the server is `local`, if you're running a server on a port that is **exposed to a public network** (e.g., not behind a firewall or not using `localhost`), be **cautious**.
- Ports like `80`, `443`, or others exposed to the web could be **vulnerable** to attacks if the server is **misconfigured**.



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}