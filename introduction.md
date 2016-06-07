Introduction
==

The goal of this book is to accommodate the reader with radare2, which is quickly becoming a bread & butter tool for any reverse engineer, malware analyst or biweekly CTF player. It is not meant to replace the [Radare2 Book](https://www.gitbook.com/book/radare/radare2book/details), but rather to complement it.

Please note that I am by no means more than a mere beginner and amateur. I have some previous experience with other tools, such as [IDA](https://www.hex-rays.com/products/ida/), GDB (with the exceptional [PEDA](https://github.com/longld/peda) extension) and [Hopper](http://www.hopperapp.com/).

This "book"s philosophy is one of "learn by doing", with occasional pause for reflection, hints and explanations. Hence, it will be organized in a series of easy-to-follow tutorials which should cover all the basic blocks required for one to do whatever he needs.

# Prerequisites

Please note that even though **radare2** is capable of glorious and esoteric things, at the end of the day it is just a tool in your reverse engineering kit. It will **not** show you or teach you:

* How computers work.
* Exploitation techniques.
* Anti-debugging mitigation.
* How to solve problems.

Ultimately, it is up to you how you use radare2 and integrate it in your workflow. Use it for just for binary diffing, numerical conversions, DNA sequencing, editing text files...

# Why radare2

Note that these are my personal reasons for choosing to dedicate some of my time to studying and documenting my findings, and trying to convey them to you, more so than anything else.

1. **It's free**. Reverse engineers have plenty of tools to choose from, yet most of them are prohibitive from a pricing standpoint alone, while others are fairly limited. How does one become a hobbyist reverse engineer?
2. **It's very actively developed**. This is a very convincing pulse, beating at 20 commits per day at times. Even if you will never report a bug, the fact that there is an almost immediate feedback from the developers gives them great credit and respect.
3. **It's versatile**. I usually abide by the saying *"do one thing and do it well"*. I have to make an exception for radare2. Even though it's still unreleased, so to speak (at the time of writing) and the official documentation isn't ready yet, it can perform surprisingly good in a variety of situations.
