---
title: "Not all machines are pingable"
date: 2019-12-12T22:16:35+05:30
---

To check if a machine(IP) is reachable, I only used `ping`. If ping is not working, then I will come to a conclusion that `IP is not reachable`. If you are like me, please read further.

#### How Ping works?
Ping sends the `ICMP Echo` to the given IP and waits for the `ICMP Echo Reply`. If we get the `Echo Reply`, then ICMP server in the given IP is able to send response to our `Echo` request. We can have timeouts for this roundtrip.

So if we didn't get `Echo Reply` can we assume that given IP is not reachable. **No, it is not true**.

#### Reasons for ping not working:
    1) ping command has been disabled for that network (by the syadmin)
    2) You have no Internet connection/network connectivity
    3) A firewall can filter ICMP Echo messages
    4) IP is not actually reachable

#### Then, how to test the reachability of an IP
    1) If you know the port,
        1) `curl IP:PORT` (hope, get call shouldn't be a problem) (Layer 7)
        2) `telnet IP PORT` once connected, we cannot disconnect (Layer 4)
    2) If you don't know the port,
        1) `traceroute IP`
