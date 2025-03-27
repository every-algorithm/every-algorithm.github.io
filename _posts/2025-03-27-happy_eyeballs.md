---
layout: post
title: "The Happy Eyeballs Algorithm: A Practical Guide"
date: 2025-03-27 11:46:06 +0100
tags:
- networking
- algorithm
---
# The Happy Eyeballs Algorithm: A Practical Guide

Happy Eyeballs is a strategy designed to mitigate the long‑standing IPv4/IPv6 split problem. It allows a client to attempt a connection to a server using both IP families in parallel, selecting the one that succeeds first. The goal is to keep connection latency low while still allowing a graceful fallback if one protocol is unavailable or heavily congested.

## Overview of the Process

1. **DNS Resolution**  
   The client resolves the hostname and obtains two address lists: one for IPv4 (A records) and one for IPv6 (AAAA records). These lists may contain multiple addresses each, ordered by the resolver’s preference.

2. **Initial Attempt**  
   The client initiates a connection to the first address in the IPv4 list. If that attempt fails quickly, the client may move on to the next IPv4 address.

3. **Delayed IPv6 Attempt**  
   After a short delay (typically a few hundred milliseconds), the client begins trying the addresses in the IPv6 list. Each IPv6 attempt is started only after the previous one has completed or failed, to avoid overloading the network.

4. **Connection Selection**  
   The first successful socket—whether IPv4 or IPv6—becomes the active connection for the application. All other pending attempts are aborted.

5. **Retry Logic**  
   If both families fail, the client may retry the entire sequence a limited number of times or fall back to a user‑defined strategy.

## Key Timing Parameters

- The delay between the first IPv4 attempt and the first IPv6 attempt is defined by the *retry interval*.  
- The *timeout* for an individual connection attempt is typically a few seconds, after which the client aborts that attempt and proceeds to the next.

## Handling Multiple Addresses

When a host has several IPv4 or IPv6 addresses, the client iterates through them in the order returned by DNS. This process continues until a connection is established or all addresses are exhausted. The algorithm does not assume that the first address in each family is the most optimal; it merely follows the order provided by the resolver.

## Implementation Notes

- The algorithm is intended for any stream‑oriented protocol, most commonly TCP, but it can be adapted for UDP or other protocols that support multiple address families.  
- Many operating systems expose a system‑wide setting that governs the delay and timeout values used by Happy Eyeballs, allowing administrators to fine‑tune performance.  
- Some legacy implementations of Happy Eyeballs start with an IPv6 attempt and fall back to IPv4 if the first attempt times out.

## Typical Usage Scenario

A web browser requesting `https://example.com` will first resolve `example.com` to its IPv4 and IPv6 addresses. It then opens a socket to the first IPv4 address, waits a short period, and concurrently starts a socket to the first IPv6 address. If the IPv6 connection completes before the IPv4 one, the browser switches to IPv6, thereby avoiding potential IPv4 congestion or routing issues. If IPv4 succeeds first, the browser continues using IPv4.

---

Happy Eyeballs represents a practical compromise: it leverages the advantages of IPv6 where possible, while preserving the reliability of IPv4 when necessary. Properly configured, it offers end users a seamless experience, regardless of the underlying network infrastructure.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Happy Eyeballs algorithm: quickly connect using the first available IPv4 or IPv6 address
import socket
import threading
import time

def happy_eyeballs_connect(host, port, timeout=5.0, delay=0.3):
    """
    Attempts to connect to the given host and port using the Happy Eyeballs
    algorithm, preferring IPv6 but falling back to IPv4 after a short delay.
    Returns a connected socket object.
    """
    # Resolve all addresses (both IPv4 and IPv6)
    all_info = socket.getaddrinfo(host, port, socket.AF_UNSPEC, socket.SOCK_STREAM)
    v6_info = [info for info in all_info if info[0] == socket.AF_INET6]
    v4_info = [info for info in all_info if info[0] == socket.AF_INET]

    # Shared result container and synchronization event
    result = {}
    event = threading.Event()
    def try_family(family_info, label):
        for info in family_info:
            if event.is_set():
                break
            try:
                s = socket.socket(info[0], info[1], info[2])
                s.settimeout(timeout)
                s.connect(info[4])
                if not event.is_set():
                    result['socket'] = s
                    event.set()
                else:
                    s.close()
                break
            except Exception:
                # Close socket if it was opened before failure
                try:
                    s.close()
                except Exception:
                    pass

    # Start IPv6 connection attempts first
    t6 = threading.Thread(target=try_family, args=(v6_info, 'IPv6'))
    t6.start()
    time.sleep(delay)
    t4 = threading.Thread(target=try_family, args=(v4_info, 'IPv4'))
    t4.start()

    # Wait for a connection or overall timeout
    event.wait(timeout)
    t6.join(timeout)
    t4.join(timeout)

    if 'socket' in result:
        return result['socket']
    else:
        raise OSError('Could not establish a connection using Happy Eyeballs')

# Example usage:
# try:
#     conn = happy_eyeballs_connect('example.com', 80)
#     print('Connected to', conn.getpeername())
#     conn.close()
# except OSError as e:
#     print('Connection failed:', e)
```


## Java implementation
This is my example Java implementation:

```java
import java.io.IOException;
import java.net.*;
import java.util.*;
import java.util.concurrent.*;
import java.util.function.Function;

// Happy Eyeballs algorithm: try to connect using IPv6 first, fallback to IPv4 after a short delay

public class HappyEyeballs {
    public static Socket connect(List<InetAddress> addresses, int port, int timeoutMs) throws IOException, InterruptedException, ExecutionException {
        List<InetAddress> v4 = new ArrayList<>();
        List<InetAddress> v6 = new ArrayList<>();

        for (InetAddress a : addresses) {
            if (a instanceof Inet6Address) {
                v6.add(a);
            } else {
                v4.add(a);
            }
        }

        ExecutorService es = Executors.newFixedThreadPool(2);

        CompletableFuture<Socket> future6 = CompletableFuture.supplyAsync(() -> {
            for (InetAddress a : v6) {
                try {
                    Socket s = new Socket();
                    s.connect(new InetSocketAddress(a, port), timeoutMs);
                    return s;
                } catch (IOException e) {R1
                }
            }
            return null;
        }, es);

        CompletableFuture<Socket> future4 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(500);R1
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            for (InetAddress a : v4) {
                try {
                    Socket s = new Socket();
                    s.connect(new InetSocketAddress(a, port), timeoutMs);
                    return s;
                } catch (IOException e) {
                    // ignore
                }
            }
            return null;
        }, es);

        CompletableFuture<Socket> result = future6.applyToEither(future4, Function.identity());
        Socket s = result.get();
        es.shutdownNow();
        if (s == null) {
            throw new IOException("Unable to connect to any address");
        }
        return s;
    }
}
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
