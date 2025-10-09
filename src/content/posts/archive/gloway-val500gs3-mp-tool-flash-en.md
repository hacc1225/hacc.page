---
title: Re-flashing the Gloway VAL500GS3 Firmware
published: 2018-04-05T10:56:00Z
description: ''
updated: ''
tags:
  - archive
draft: false
pin: 0
toc: true
lang: 'en'
abbrlink: 'gloway-val500gs3-mp-tool-flash'
---

The Gloway VAL500GS3 is a solid-state drive that uses the M.2 interface and features the SM2246EN controller.

Here is the process to get the drive into a state where it can be detected by the MP Tool:

Short the JP1-1 pins → Power on the drive → Remove the jumper → Open the MP Tool → Click 'Scan' → Drive is detected

![](../_images/IMG_20180123_150004_guetzli.jpg)

For the rest of the re-flashing process, refer to any generic tutorial for the SM2246EN controller.
