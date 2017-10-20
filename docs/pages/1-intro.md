---
layout: default
title: Introduction
pdf: true
permalink: /intro
toc: true
---

# Introduction

For quite some time, our unit of understanding of computer systems has been based on the operating system. It is the level of magnification at which we operate (personal computers) and the starting point for working with data, software, and services. With the rising popularity of Linux containers, we have carried forward that unit of operation to be the primary content of containers. While this practice affords reproducibility by way of providing encapsulated, portable environments, in the same way that we cannot reliably predict how to call a program on a colleague's computer without having outside knowledge, the contents of the containers are not internally modular or programmatically understandable. For the scientific community, this is a barrier to providing scientific containers that are both modular and programmatically accessible. Scientists need to not only encapsulate environments and software in reproducible containers, they need to make that content easily discoverable.

To motivate the need for modularity, let’s imagine a scenario where a researcher is developing a container to serve software to answer a specific scientific question. Let’s reduce the complexity of the container, and say that all dependencies aside from a main executable come with the base operating system of the container. Even under this use case, we face a significant issue: the community that wants modularity and programmatic accessibility (developers and maintainers of infrastructure) are usually not the producers of the scientific software. The scientist creating the software, without any help, is likely to produce a container that is akin to a black box. 

<img src="/SCI-F/img/robot79.png" width="50%"/>

He or she will likely use the container with an understanding of its structure, and not tell others that, for example, the application expects its data input at some `/data` inside the container, or that for a slightly different command, a second executable is provided that must be called directly. We cannot have knowledge about where the software is installed, how to run it, or possibly integrate it with other resources. 

In the *best* case scenario, executing the container will execute some main driver of the software. This simple usage model breaks when there is more than one executable provided. Can the drivers be predictably found? If a particular directory is bound from the host, is any important container content lost? Are they isolated to allow for easy comparison? This lack of internal modularity is the first compelling need for such a standard. It must be easy for the creator to define and organize multiple applications within a single container.

The next need is driven by a larger goal of encouraging reproducible practices. At first glance, containers are a leap in the right direction. Given that all dependencies are packaged nicely, a container is very reproducible. The single container gives the user absolutely everything - a complete operating system with data, libraries, and software. But this also means that we are producing heavy containers to serve a small amount of software. In terms of reproducibility, we have lost modularity because "best practice" means there is no easy way to untangle the scientist's contribution to the container from the base. "Best practice" of a single container is encouraging production of many similar black boxes that are difficult to compare. There are several problems with this practice:

 * Containers are not **consistent** to allow for comparison. Two containers with the same software installed in different locations do not obviously do the same thing, despite this being a possibility.

 * Containers are not **transparent**. If i discover a container and do not have any prior knowledge or metadata, a known function may be completely concealed.

 * Container contents are not easily available for **introspection**, or programmatically understandable. I should be able to run a function over a container, and know exactly the functions available to me, ask for help for a function, or how and where to interact with inputs and outputs.

 * Container internal infrastructure is not **modular**. We would be weary to export an entire container into another because of overlapping content.

The basis of these core problems can be reduced to the fact that we are being forced to operate on a level that no longer makes sense given the needs of the problem. We need to optimize definition and modular organization of applications and data, and this is a different set of goals than structuring one system per the <a href="https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard" target="_blank">Filesystem Hierarchy Standard</a>. This goal is also not met moving up one level to one software package per container, because there is huge redundancy with regard to the duplicated filesystem, and what has been added and is now hiding amongst it.. 

The above problems also hint that the generation of containers is not easy. When a scientist starts to write a recipe for his set of tools, he probably doesn't know where to put it, perhaps that a help file should exist, or that metadata about the software should be served by the container. If container generation software made it easy to organize and capture container content automatically, we would easily meet these goals of internal modularity and consistency, and generate containers that easily integrate with external hosts, data, and other containers.

Based on these problems, it is clear that we need direction and guidance on how organization multiple applications and data within a single container in a way that affords modularity, programmatic accessibility, transparency, and consistency. This document will review the rationale and use cases for the Standard Container Integration Format (SCI-F). We will first review the goals of the architecture, followed by it's primary use case: easy integration with tools that allow for organization and comparison. For interested readers, we finish with some background on its development, and future analysis and work that is afforded by it. This document that describes the specification for SCI-F is openly available for critique and contribution at http://containers-ftw.org/SCI-F/, and as a community we adhere to the <a href="https://open-stand.org/about-us/principles/" target="_blank">Open Stand Principes</a>.


## Goals

The Standard Container Integration Format (SCI-F) establishes an overall goal to make containers consistent, transparent, parseable, and internally modular.  Using Singularity, we start with a reproducible encapsulated environment that includes packaged software and data modules. SCI-F improves upon this base. We assert that for a container to conforms to SCI-F, it must:

 - be **consistent** to allow for comparison. I am able to easily discover relevant software and data for one or more applications defined by the container creator.
 - be **transparent**. If I discover a container and do not have any prior knowledge or metadata, the important executables and metadata are revealed to me.
- Container contents are easily available for **introspection** because they are programmatically parseable. I can run a function over a container, and know exactly the functions available to me, ask for help, and know where to interact with inputs and outputs.
 - Container internal infrastructure is **modular**. Given a set of SCI-F apps from different sources, I can import different install routines and have assurance that environment variables defined for each are sourced correctly for each, and that associated content does not overwrite previous content. Each software and data module must carry, minimally, a unique name and install location in the system.

To be clear, this is **not** a specification for a <a href="https://github.com/opencontainers/image-spec/blob/master/spec.md" target="_blank">container image</a>, or a <a href="https://github.com/pditommaso/awesome-pipeline" target="_blank">workflow using containers </a>. Although these goals match nicely with efforts for workflow and image standardization, SCI-F is a specification for modular organization of *content* within the image, untangled from the file system and image itself. Any container that implements SCI-F to achieve the goals of consistency, transparency, and modularity must provide an easy means to adopt this internal structure and expose it for use. We first introduce the idea of container "apps" that are installed easily, and conform to a predictable internal organization. Each of the specific goals in context of the assertions is discussed in more detail in the following sections.


### Consistency

Given the case of two containers with the same software installed, it should be the case that the software is always found in the same location. Further, it should be the case that the data (inputs and outputs) that is used and generated at runtime is also located in a consistent manner. To achieve this goal, SCI-F defines a new root folder,  `/scif`, chosen to be named to have minimal conflict with existing cluster resources. Under this folder are separate folders for each of software modules (`/scif/apps`), and data (`/scif/data`) where the container generation software should generate subfolders for each modular application installed. For example, a container with applications foo and bar would have them installed as follows:

```
/scif
    /apps
        /bar
        /foo
```

Thus, if two containers both have `foo` installed, we would know to find the installation under `/scif/apps/foo`. Data takes a similar approach. We define a new root folder, `/scif/data`, with a similar subfolder organization:

```
/scif
    /data
       /bar
       /foo
```

Thus, a container in a workflow that knows to execute the foo application would also know where to direct output, or find inputs. The details of the contents of these directories will be discussed further.


### Transparency

Arguably, when we want to know about a container's intended use, we don't care so much for the underlying operating system. We would want to subtract out this base first, and then glimpse at what remains. Given the consistent organization above, and importantly, a distinction between container base operating system (for example, software in `/bin`, `/sbin`, or even sometimes `/opt`, we can easily determine the container's additional software with a simple list command:

```
singularity exec container.img ls /scif/apps -l

  bar
  foo
```

And in the case of the Singularity implementation, this information is available with a shorthand flag, apps. 


```
singularity apps containers.img
  
  bar
  foo
```

We can predictably find and investigate a particular software given that we know the name:

````
singularity shell --pwd /scif/apps/foo container.img
 
$ echo $PWD
$ /scif/apps/foo
```

Or ask a container to run a specific software:

```
singularity run --app foo container.img
RUNNING FOO
```

Another reason to advocate for an organization that is different from most base operating systems is because of mounting. A cluster onto which a container is mounted should be able to (in advance) know the paths that are allowed (`/scif/apps`) and (`/scif/data`) and not have these interfere with possibly software that already exists (and might be needed) on the cluster (e.g., `/opt`). From a high level, we are introducing a simply formatted addition to the standard Linux File System, a folder at the root that starts with `/scif` to confidently say "the content under this location is exclusively relevant to the purpose of the container."

### Parsability

Parsability comes down to programmatic accessibility. This means that, for each software module installed, we need to be able to (easily) do the following:

- **provide an entrypoint** (e.g., the current "runscript" for a Singularity container, or the Dockerfile `ENTRYPOINT` and `CMD` serve this purpose). We arguably need the different software modules within the container to each provide an entrypoint.
- **provide help**. Given an entrypoint, or if a user wants to understand an installed application, it should be the case that a user can issue a command to view documentation provided for the software. For the developer, adding this functionality should be no harder than writing text in a section of a file.
- **provide metadata**. A software module might have a version, a point of contact or link to further documentation, an author list, or other important metadata values that should be (also) programmatically accessible.

SCI-F will accomplish these goals by creating a metadata folder to serves each software module installed within the container. In the case of the Singularity implementation, this metadata folder mirrors the container's global metadata folder.
 

### Modularity

A container with distinct, predictable locations for software modules and data is modular. The organization of these modules under a common root ensures that each carries a unique name. Further, this structure allows for easy movement of modules between containers. If a modular carries with it information about installation and dependencies, it could easily be installed in another container. The user doesn't need to look through the tangle of a single build recipe (e.g., a Dockerfile or `%post` section in a Singularity recipe) to find a function of interest.


<div>
    <a href="/SCI-F/"><button class="previous-button btn btn-primary"><i class="fa fa-chevron-left"></i> </button></a>
    <a href="/SCI-F/modules.html"><button class="next-button btn btn-primary"><i class="fa fa-chevron-right"></i> </button></a>
</div><br>
