---
layout: post
title:  "Swarm Cluster on ARM SBCs"
date:   2025-05-05 16:59:28 -0300
categories: jekyll update
image: /assets/sbcsinfos.png
---

This is simply a doc/work-log of my setup for a Docker Swarm with load balancing using 4 single-board computers (SBCs): a Banana Pi M2 Zero W, Orange Pi Zero 2W, Raspberry Pi 3B, and Raspberry Pi Zero W.

![]({{ page.image }})

All but the Raspberry Pi 3B are immersed in direct liquid cooling using mineral oil (I think it's cool — I like it 😄).

The idea behind this is that I got a bit obsessed with power usage. I wanted to try different solutions that might perform well. I mean, it's better to have an SBC as your always-on server than an old notebook running 24/7. Even if you put the notebook to sleep when idle, it still uses a fair amount of electricity.

In contrast, these SBCs consume very little power and can run a surprising number of things. Of course, many applications *can’t* run on ARM architecture, which is another topic — but what can I do? I already have the boards, so I want to test them 😅

The goal here is to set up a Docker Swarm cluster, enable load balancing (if possible), run some stress tests, and measure power usage and thermals. I’ll then compare all that to just using my old notebook as a server — maybe I can finally sell it after this experiment!

