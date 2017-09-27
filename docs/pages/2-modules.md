---
layout: default
title: Container Modules
pdf: true
permalink: /modules
---


# Modules

Modularity can be understood as the level of dimensionality that a user is instructed to operate, and for the purposes of this discussion we will suggest three general levels. 

 - **Node** For those familiar with container technology, it is commonly the case that an entire container is considered a module. An example is a container that performs the task of variant calling. If the container itself is considered the module, the user would expect to provide raw data inputs, and receive final results as an output. The container acts as a node that plugs into higher level orchestration tools. The node representation is ideal if the container is expected to plug into a workflow manager and perform one task.
 - **Internal**: A second common scenario might be a single container that holds executables to perform different steps of a pipeline, perhaps so that the researcher can use the same container to run multiple steps, or perform any number of steps in parallel. This container would come with multiple *internal modules*, each performing a series of commands for one step in the pipeline (e.g., the step "mapping" uses internal commands from software `bwa` and `samtools`). The user doesn't need to know the specifics of the steps, but how to call them. We call this level "internal modules" because without any formal structure for the contents of containers, they are hidden, internal executables that must be found or described manually.
 - **Development**: Containers can also serve modules that are represented at the ideal level for development. For this example, instead of providing the container as a node, or actions inside like "mapping", the smallest units of software are exposed, such as the executables `bwa` and `samtools`. It would be likely that a researcher developing a scientific pipeline would find this useful.

Given the different needs briefly explained above, it is clear that there is no correct level of dimensionality to define a module. 

>> The definition of modularity is entirely based on the needs of the creator and user. 


If we discover a container after creation, it cannot be clear without suitable documentation what level is represented, or how to interact with the container. What is needed is an ability for the creator of a container to implicitly define this level of usage simply by way of creating the container. SCI-F allows us to do this. We can define modules on the levels of single files, or groups of software to perform a task. The metadata and organization of our preferences is automatically generated to create a complete, and programmatically understandable software package or scientific analysis.

<br>
<div>
    <a href="/SCI-F/intro.html"><button class="previous-button btn btn-primary"><i class="fa fa-chevron-left"></i> </button></a>
    <a href="/SCI-F/practices.html"><button class="next-button btn btn-primary"><i class="fa fa-chevron-right"></i> </button></a>
</div><br>
