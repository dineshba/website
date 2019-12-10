---
title: "Resolve_vs_Reach"
date: 2019-12-10T08:46:06+05:30
---

I have seen lot of people confuse between resolving dns vs reach IP. So this small blog.

Inorder to get data from any computer we need to know the address of the computer which is IP address. Let's say we want to get some data from google and its IP is `172.217.31.196` (currently). Its very hard to remember this IP and also they can have multiple machines(IPs) which will give the same data (horizontal scaling). So we cannot remember IPs. So we need a map (key: name, value: IP) stored in a server called DNS whose IP is configured automatically (done by DHCP server) when we connect to internet (8.8.8.8 or 1.1.1.1 or your custom dns server). 

So when we do `www.google.com` in browser
    
    1) Check which dns server to use in `/etc/resolve.conf`
    2) Use that dns server to get the `ip address` of www.google.com
    3) Use that IP address to connect and transfer data

**First two steps** is `resolve` and **third step** is `reach`.


### Example:
Let's say I want to expose/transfer some data to my colleagues. And my ip is `192.168.1.4` which is private and will be accessible only to the people whose is in my network. Instead of asking my colleagues to use this IP, we can put an entry in public DNS server (8.8.8.8 or 1.1.1.1). Key is `www.dineshba.com` and value is `192.168.1.4`. Once that is done, anybody can **resolve** `www.dineshba.com` to `192.168.1.4` as we added entry in public dns server. But it is reachable only to people in my network.