---
layout: single
title: Write Your Own DROP Rule
permalink: /iptables/write-your-own-drop-rule
sidebar:
  title: "Iptables"
  nav: sidebar-iptables
---

## Assumptions
- You have the docker CLI installed

## What
Experiment with writing your own iptables rules within the safety of your own
docker container.

## How

📝 **Get your docker container setup**
0. Run an ubuntu docker container and attach to it:
  ```
  docker run --privileged -it ubuntu bin/bash
  ```
0. Set up the docker container
  ```
  apt-get update
  apt-get install iptables
  apt-get install curl
  ```

📝 **What's the current state of the world?**
0. Look at the default iptables rules
  ```
  iptables -S
  ```
It should look like this:
  ```
  -P INPUT ACCEPT
  -P FORWARD ACCEPT
  -P OUTPUT ACCEPT
  ```
Input, forward, and output are all names of chains. Currently there are no
rules attached to these chains. What beautiful empty chains! This means that
traffic is not being altered by iptables.

0. See that traffic is not being restricted by running `curl google.com`.  It
   works!

📝 **Make your own DROP rule**

Let's make a rule to DROP all traffic from the container so that the curl will
fail.

0. Make your own custom chain.
```
iptables -N drop-everything
```
0. Append a rule to your chain.
```
iptables -A drop-everything -j DROP
```
0. View your handiwork. Oooooh. Ahhhhhhh.
```
iptables -S
```
0. See if you can still `curl google.com`. What! The curl still works!
That's because currently nothing is hitting your rule. You need to attach your custom chain to the INPUT, FORWARD, and/or OUTPUT chain in order for traffic to hit it.

The INPUT, FORWARD, and OUTPUT chains are hit in different situations. (See diagram below)
- The INPUT chain is hit by ingress traffic (remember, the traffic is coming *in*).
- The FORWARD chain is hit by East/West traffic (remember... idk for this one).
- The OUTPUT chain is hit by egress traffic (remember, the traffic is *e*xiting the container and going *out*).

![iptables chains and tables diagram](https://storage.googleapis.com/cf-networking-onboarding-images/iptables-tables-and-chains-diagram.png)

0. The request to google is egress traffic, so we want to attach out custom chain to the OUTPUT chain.
```
iptables -A OUTPUT -j drop-everything
```

0. See if you can still `curl google.com`. You should see the error `Could not resolve host: google.com`.
0. Delete all of the rules and the chain that you created.
Before you can delete the chain itself, you need to delete the rules attached to it using the `-D` flag.
```
iptables -D EITHER-INPUT-FORWARD-OR-OUTPUT -j drop-everything
iptables -D drop-everything -j DROP
```
Then you can delete the chain itself using the `-X` flag
```
iptables -X drop-everything
```

Don't leave your docker container yet! You're going to need it in the next story!

### Expected Result

You should know how to...
- add/remove a iptables chain on a particular table
- add/remove a rule to an iptables chain

### Resources
* [iptables man page](http://ipset.netfilter.org/iptables.man.html)
* [Julia Evans iptables basics](https://jvns.ca/blog/2017/06/07/iptables-basics/)
* [iptables primer](https://danielmiessler.com/study/iptables/)
