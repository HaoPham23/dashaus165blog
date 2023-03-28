---
title: "Cyber Apocalypse 2023: Critical Flight"
subtitle:
date: 2023-03-28T22:53:07+07:00
draft: false
description:
keywords:
license:
comment: false
weight: 0
tags:
  - writeups
  - hardware
  - htb
categories:
  - Writeups
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---
A writeup on Critical Flight
<!--more-->

{{< admonition note "Challenge Information" >}}
* **Given materials:** [Get it here!](https://drive.google.com/file/d/1wPLpM6tLlZzncKIVRhR4YkUFW9G8EQl9/view?usp=sharing)
* **Description:**
* **Category:** Hardware - Very Easy
{{< /admonition >}}

## Problem statement
Given a lot of GBR file. Our mission is somehow finding the flag :D. 

## Results
These files are called Gerber files - a standard file format used in the manufacturing of printed circuit boards (PCBs) to describe the PCB's copper layers, solder mask, legend, and other features. To open this, reader can access this website: [https://www.pcbway.com/project/OnlineGerberViewer.html](https://www.pcbway.com/project/OnlineGerberViewer.html). We can easily find all parts of the flag in this board:
<img src='flag.png' alt="First part" width="1000"/>


