---
layout: post
title: "Experiment selinux"
date:  2017-06-24 14:14:00 -0700
categories: linux
---
# About Selinux
selinux is a monotonic security module in linux. We write some rules using the policy language in
user space, compile that into a binary format. Then load it into linux kernel. Then linux kernel
forces all access following the rules.

The policy language is [here](https://www.nsa.gov/resources/everyone/digital-media-center/publications/research-papers/assets/files/configuring-selinux-policy-report.pdf).

The userspace tools is [here](https://github.com/SELinuxProject/selinux).
Inside libselinux:

    `checkpolicy` is used to compile policy languages.
    `load_policy` is used to load compiled policy into kernel.
    `setfiles` is used to label file systems with selinux user:role:type labels.

An example of writing policy is [here](https://github.com/yabincui/linux_kernel_exp/blob/master/sepolicy/gen_policy.py).

An example of labeling filesystems using `setfiles` is [here](https://github.com/yabincui/linux_kernel_exp/blob/master/make_image.py).
