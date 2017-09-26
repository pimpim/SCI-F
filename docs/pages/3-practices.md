---
layout: default
title: Reproducible Practices
pdf: true
permalink: /practices
toc: true
---

# Practices

It is important to make the distinction between the reproducibility of an entire container, and a software module installed under `/scif/apps`. While the container itself is portable, and designed to contain all dependencies to support reproducibility, the SCi-F module in and of itself is not guaranteed to be. For example, a user might define a  module only with an `%apprun` section, implying that the folder only contains a runscript to execute. The user may have chosen to install dependencies for this script globally in the container, in the `%post` section, because perhaps they are shared across multiple modules. Under these conditions, if another user expected to add the module to a different build recipe, the dependencies from `%post` would be needed too. The host operating system also needs to be taken into consideration. A module with dependencies installed from the package manager "yum" would not move seamlessly into a debian base. However, appropriate checks and balances can be implemented into the process of moving applications:

 - For applications that must be portable outside of their initial container, users would be encouraged to include all dependency installs within the `%appinstall` section. If they were already installed during `%post`, the package would be found and skipped.
 - Installing an application into a container would check for OS compatibility. This can be done automatically by storing information about the base OS with each application as a label. To encourage this practice, we have added a test and requirements of specifying one or more operating systems for any module contributed at <a href="https://containers-ftw.github.io/apps" target="_blank">https://containers-ftw.github.io/apps</a>. With these checks, we can have some confidence that the recipes for generating the apps are maximally portable.

Modular internal contents combined with reproducible portable environments via Singularity containers is a starting point for practicing good science.
