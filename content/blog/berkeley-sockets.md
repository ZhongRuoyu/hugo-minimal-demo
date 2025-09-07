---
title: Berkeley Sockets
date: 2022-02-16
author: Wikipedia
summary: Berkeley sockets is an application programming interface (API) for Internet sockets and Unix domain sockets, used for inter-process communication (IPC).
---

(Extracted from [Berkeley sockets - Wikipedia](https://en.wikipedia.org/wiki/Berkeley_sockets). Licensed under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/).)

## Header Files

The Berkeley socket interface is defined in several header files. The names and content of these files differ slightly between implementations. In general, they include:

| File           | Description                                                                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `sys/socket.h` | Core socket functions and data structures.                                                                                                                               |
| `netinet/in.h` | AF_INET and AF_INET6 address families and their corresponding protocol families, PF_INET and PF_INET6. These include standard IP addresses and TCP and UDP port numbers. |
| `sys/un.h`     | PF_UNIX and PF_LOCAL address family. Used for local communication between programs running on the same computer.                                                         |
| `arpa/inet.h`  | Functions for manipulating numeric IP addresses.                                                                                                                         |
| `netdb.h`      | Functions for translating protocol names and host names into numeric addresses. Searches local data as well as name services.                                            |

## Client-server example using TCP

### Server

Establishing a TCP server involves the following basic steps:

- Creating a TCP socket with a call to {{< highlight c "hl_inline=true" >}}socket(){{< /highlight >}}.
- Binding the socket to the listening port ({{< highlight c "hl_inline=true" >}}bind(){{< /highlight >}}) after setting the port number.
- Preparing the socket to listen for connections (making it a listening socket), with a call to {{< highlight c "hl_inline=true" >}}listen(){{< /highlight >}}.
- Accepting incoming connections ({{< highlight c "hl_inline=true" >}}accept(){{< /highlight >}}). This blocks the process until an incoming connection is received, and returns a socket descriptor for the accepted connection. The initial descriptor remains a listening descriptor, and {{< highlight c "hl_inline=true" >}}accept(){{< /highlight >}} can be called again at any time with this socket, until it is closed.
- Communicating with the remote host with the API functions {{< highlight c "hl_inline=true" >}}send(){{< /highlight >}} and {{< highlight c "hl_inline=true" >}}recv(){{< /highlight >}}, as well as with the general-purpose functions {{< highlight c "hl_inline=true" >}}write(){{< /highlight >}} and {{< highlight c "hl_inline=true" >}}read(){{< /highlight >}}.
- Closing each socket that was opened after use with function {{< highlight c "hl_inline=true" >}}close(){{< /highlight >}}

The following program creates a TCP server listening on port number 1100:

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(void)
{
  struct sockaddr_in sa;
  int SocketFD = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
  if (SocketFD == -1) {
    perror("cannot create socket");
    exit(EXIT_FAILURE);
  }

  memset(&sa, 0, sizeof sa);

  sa.sin_family = AF_INET;
  sa.sin_port = htons(1100);
  sa.sin_addr.s_addr = htonl(INADDR_ANY);

  if (bind(SocketFD,(struct sockaddr *)&sa, sizeof sa) == -1) {
    perror("bind failed");
    close(SocketFD);
    exit(EXIT_FAILURE);
  }

  if (listen(SocketFD, 10) == -1) {
    perror("listen failed");
    close(SocketFD);
    exit(EXIT_FAILURE);
  }

  for (;;) {
    int ConnectedFD = accept(SocketFD, NULL, NULL);

    if (0 > ConnectedFD) {
      perror("accept failed");
      close(SocketFD);
      exit(EXIT_FAILURE);
    }

    /* perform read write operations ...
       ...
       ... */

    if (shutdown(ConnectedFD, SHUT_RDWR) == -1) {
      perror("shutdown failed");
      close(ConnectedFD);
      close(SocketFD);
      exit(EXIT_FAILURE);
    }
    close(ConnectedFD);
  }

  close(SocketFD);
  return EXIT_SUCCESS;
}
```

### Client

The following program creates a connection to a TCP server that is listening on port number 1100 at address 127.0.0.1:

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(void)
{
  struct sockaddr_in sa;
  int res;
  int SocketFD = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
  if (SocketFD == -1) {
    perror("cannot create socket");
    exit(EXIT_FAILURE);
  }

  memset(&sa, 0, sizeof sa);

  sa.sin_family = AF_INET;
  sa.sin_port = htons(1100);
  res = inet_pton(AF_INET, "127.0.0.1", &sa.sin_addr);

  if (connect(SocketFD, (struct sockaddr *)&sa, sizeof sa) == -1) {
    perror("connect failed");
    close(SocketFD);
    exit(EXIT_FAILURE);
  }

  /* perform read write operations ...
     ...
     ... */

  shutdown(SocketFD, SHUT_RDWR);
  close(SocketFD);
  return EXIT_SUCCESS;
}
```
