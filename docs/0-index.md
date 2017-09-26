---
layout: default
title: {{ site.name }}
pdf: true
permalink: /
---

<div style="float:right; margin-bottom:50px; color:#666">
Version: {{ site.version }}<br>
Date: 2017-xx-xx
</div>

# Abstract
Here we present the Standard Container Integration Format (SCI-F), an organizational format for internally modular scientific containers that easily expose executables and metadata for discoverability and integration. With SCI-F, a published scientific workflow can have multiple entry points to serve reproducible, modular pipeline steps, each that includes its own environment, metadata, installation steps, and a primary executable script. We will start by reviewing the background and rationale for a container organizational format, and how SCi-F achieves the goals of modularity, transparency, and consistency. We then review the organizational structure of the standard, and the different levels of internal modules that it affords. For this work, we have implemented and released the format with the Singularity (Kurtzer et al.) software version 2.4, and discuss how a user would interact with a container generated with SCI-F. Finally, we discuss implemented use cases for SCI-F to evaluate container software, provide metrics, serve scientific workflows, and execute a primary function under different contexts. To encourage collaboration and sharing of apps, we have developed an open source, version controlled, tested, and programmatically accessible web infrastructure at <a href="http://containers-ftw.org/apps" target="_blank">http://containers-ftw.org/apps</a>. The ease of using SCI-F to develop scientific containers offers promise for scientists to easily generate self-documenting containers that are programmatically parseable, exposing software and associated metadata, environments, and files to be quickly found and used.

What are you waiting for? Start out with the [introduction](/SCI-F/intro.html).

<div>
    <a href="/SCI-F/intro.html"><button class="next-button btn btn-primary"><i class="fa fa-chevron-right"></i> </button></a>
</div><br>
