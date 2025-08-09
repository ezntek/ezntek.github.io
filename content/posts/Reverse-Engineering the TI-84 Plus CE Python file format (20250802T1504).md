+++
title = "Reverse-Engineering the TI-84 Plus CE Python File Format"
date = 2025-08-02T15:04:11+08:00
author = "ezntek"
tags = [ "ti84" ]
description = "I love proprietary binary file formats that just encode ASCII~"
draft = true
+++

## Why??

There is a backstory, just like any other blog article of mine.

[Image of my new TI-84+CE Python](/img/ti84/newcalc.png)

When you give Eason a device, he will analyze it. When he sees a feature, he explores it. When he sees no manual, he checks the source code. If the program is proprietary...

*...his primal insticts kick in, and he runs `xxd`.*

### Context

The TI-84 plus CE Python, as advertised, lets you edit Python files directly on the calculator.

<script src="https://utteranc.es/client.js"
        repo="ezntek/ezntek.github.io"
        issue-term="title"
        label="comments"
        theme="github-dark"
        crossorigin="anonymous"
        async>
</script>
