---
title: Slotprovider Performance experiment
author: Christoph Portmann
date:   2017-08-20 00:00
year: 2017
weight: 0
link: https://github.com/chrisport/slotprovider
image: /images/go.png
---
Slotprovider manages a number of free slots which can be acquired and released concurrently.
No blocking: If there is no free slot, the Acquire method returns immediately with false.
This package is a playground for performance optimization.
