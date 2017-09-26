---
layout: default
title: Container Modules
pdf: true
permalink: /modules
---

# Modules

Modularity can be understood as the level of dimensionality that a user is instructed to operate, and for the purposes of this discussion we will describe three general levels. 

 - `Node` For those familiar with container technology, it is commonly the case that an entire container is considered a module. We can describe a container that performs variant calling to exemplify different levels of dimensionality. If the container itself is considered the module, the user would expect to provide raw data inputs, and receive final results as an output. The container acts as a node that plugs into higher level orchestration tools.
 - `Internal Module`: A more realistic scenario might be a single container that holds executables to perform different steps of a pipeline, perhaps so that the researcher can run steps in parallel, or in different environments. This container would come with multiple internal modules, each performing a series of commands for one step in the pipeline (e.g., mapping with software `bwa` and `samtools`). The user doesn't need to know the specifics of the steps, but how to call them. We call this level "internal modules" because without any formal structure for the contents of containers, they are hidden, internal executables that must be found or described manually.
 - `Development Module`: Containers can also serve modules that are represented at the ideal level for development. For this example, instead of providing the container as a node, or actions inside like "mapping", the smallest units of software are exposed, such as the executables `bwa` and `samtools`. It would be likely that a researcher developing a scientific pipeline would find this useful.

From the above example, we see that there is no correct level of dimensionality to define a module - it is entirely based on the needs of the creator and user. In this case, what is needed is an ability for the creator of a container to implicitly define this level of usage simply by way of creating the container. SCI-F allows us to do this. We can define modules on the levels of single files, or groups of software to perform a task. The metadata and organization of our preferences is automatically generated to create a complete, and programmatically understandable software package or scientific analysis.
