---
title: "EEM: To sync or not to sync"
date: 2022-12-21T07:01:24-06:00
draft: false
---

For those of you familiar with Cisco’s EEM (Embedded Event Manager), this post will try to explain how and when to use the “sync” parameter. It’s usage seems to confuse users, including myself sometimes. So I did some reading and testing, and here’s what I found.

Note that EEM usually requires an event detector to be specified, i.e. CLI patterns, syslog patterns, routing events, etc. The sync argument is only used with the CLI event detector.

As far as I can tell, synchronous mode (“sync yes”) is required **only** if you want your EEM applet to run **before** the original CLI command. If you don’t care about the order, or don’t want to run the original CLI command, then it doesn’t matter whether you use sync mode or not.

Let’s show some examples.

*** Synchronous mode (“sync yes”)

So you want run the applet before the matched CLI command. In this example, whenever the user types the `show alias` command, you want a `show clock` to be executed first:

```
Foo
Foobar

```


