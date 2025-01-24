# IRCSERV 

Create your own IRC server in C++, fully compatible with an official client

## Table of Contents

- [I/O Multiplexing](#io-multiplexing)
- [Socket Functions](#socket-functions)
  - [send()](#send)
  - [recv()](#recv)
  - [signal()](#signal)
  - [bind()](#bind)
  - [connect()](#connect)
  - [listen()](#listen)
  - [accept()](#accept)
  - [htons()](#htons)
  - [socket()](#socket)
  - [close()](#close)
  - [kevent()](#kevent)
- [Example Code Using I/O Multiplexing](#example-code-using-io-multiplexing)
- [Use Cases](#use-cases)

## I/O Multiplexing

**I/O Multiplexing** refers to the ability of a program to handle multiple I/O operations (such as reading from or writing to multiple sockets) at the same time. Without I/O multiplexing, a program would need to process each I/O operation one at a time, potentially blocking the entire program until an operation completes.

I/O multiplexing allows a program to wait for events on multiple I/O streams and only act when data is available, allowing for more efficient resource utilization.

### Types of I/O Multiplexing

There are several system calls available for implementing I/O multiplexing:

### select()

`select()` is one of the oldest and most widely used methods for I/O multiplexing. It allows a program to monitor multiple file descriptors (e.g., sockets) and wait for them to become ready for reading, writing, or have an exceptional condition.

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- `nfds`: The highest-numbered file descriptor in any of the sets, plus 1.
- `readfds`, `writefds`, `exceptfds`: These are sets of file descriptors to monitor for reading, writing, or exceptional conditions.
- `timeout`: A time limit on the function's operation.

### poll()

`poll()` is similar to `select()`, but it does not have the file descriptor limit of `select()`. It uses an array of `pollfd` structures to describe the file descriptors to be monitored.

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

### epoll()

`epoll()` is a highly efficient I/O multiplexing method available on Linux. It supports handling large numbers of file descriptors and offers better performance than `select()` and `poll()`.

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

### kqueue()

`kqueue()` is an I/O event notification mechanism available on BSD-based systems (including macOS). It provides an efficient way to monitor multiple file descriptors.

```c
int kqueue(void);
int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
```

## Socket Functions

### send()

The `send()` function sends data over a socket. It is often used to send data to a connected peer.

```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

### recv()

The `recv()` function receives data from a socket. It waits for data to be available and reads it into a buffer.

```c
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

### signal()

The `signal()` function is used to set up a signal handler for signals sent to a process.

```c
void (*signal(int sig, void (*handler)(int)))(int);
```

### bind()

The `bind()` function is used to associate a socket with a specific address and port.

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

### connect()

The `connect()` function establishes a connection to a remote server.

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

### listen()

The `listen()` function prepares a socket to accept incoming connections.

```c
int listen(int sockfd, int backlog);
```

### accept()

The `accept()` function accepts an incoming connection on a listening socket.

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

### htons()

The `htons()` function converts a 16-bit integer from host byte order to network byte order.

```c
uint16_t htons(uint16_t hostshort);
```

### socket()

The `socket()` function creates a new socket for communication.

```c
int socket(int domain, int type, int protocol);
```

### close()

The `close()` function closes a socket, terminating its use.

```c
int close(int fd);
```

### kevent()

The `kevent()` function is used to register or modify events on a kqueue.

```c
int kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
```

## Example Code Using I/O Multiplexing

Here's a simple example using `select()` for handling multiple client connections:

```c
#include <stdio.h>
#include <sys/socket.h>
#include <sys/select.h>
#include <unistd.h>

#define MAX_CLIENTS 10

int main() {
    int server_fd, client_fd;
    fd_set readfds;
    struct timeval timeout;

    // Initialize server socket (code omitted)
    server_fd = create_server_socket();

    // Wait for client connections using select
    while (1) {
        FD_ZERO(&readfds);
        FD_SET(server_fd, &readfds);

        // Add client sockets to the set (code omitted)
        add_client_sockets(&readfds);

        timeout.tv_sec = 5;
        timeout.tv_usec = 0;

        int activity = select(MAX_CLIENTS, &readfds, NULL, NULL, &timeout);
        if (activity > 0) {
            if (FD_ISSET(server_fd, &readfds)) {
                // Handle new client connection
                client_fd = accept_connection(server_fd);
                FD_SET(client_fd, &readfds);
            }

            // Handle communication with clients (code omitted)
        }
    }

    return 0;
}
```


