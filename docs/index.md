---
layout: default
title: {{ site.name }}
---

# Introduction

For quite some time, our unit of understanding has been based on the operating system. It is the level of magnification at which we understand data, software, and products of those two things. Recently, however, two needs have arisen:

We simeotaneously need **modularity** and **reproducible practices**. At first glance, these two things don't seem very contradictory. A modular piece of software, given that all dependencies are packaged nicely, is very reproducible. The problem arises because it's never the case that one software package can independently drive an entire solution. A single problem, whether it be sequencing genetic code, predicting prostate cancer reocurrence from highly dimensional data, or writing a bash script to play tetris, requires many disparate dependencies. Given our current level of understanding of information, the operating system, the best that we can do is give the user absolutely everything - a complete operating system with data, libraries, and software. But now for reproducibility we have lost modularity. We are producing heavy containers to serve a small amount of software, and each is a special snowflake. There are several problems with this practice:

 1. Containers are not **consistent** to allow for comparison. Two containers with the same software installed in different locations do not obviously do the same thing, despite this being a possibility.
 2. Containers are not **transparent**. If i discover a container and do not have any prior knowledge or metadata, a known function may be completely concealed.
 3. Containers are not **parseable**, or programatically understandable. I should be able to run a function over a container, and know exactly the functions available to me, ask for help for a function, or how and where to interact with inputs and outputs.
 4. Container interal infrastructure is not **modular**. We would be weary to export an entire container into another because of overlapping content.

The basis of these core problems can be reduced to the fact that we are being forced to operate on a level that no longer makes sense given the needs of the problem. We need to optimize definition and modular organization of applications and data, and this is a different set of goals than structuring one system per the [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard). This goal is also not met moving up one level to one software package per container, because there is huge redudancy with regard to the duplicated filesystem. 

The above problems also hint that the generation of containers is not easy. When a scientist starts to write a recipe for his set of tools, he probably doesn't know where to put it, perhaps that a help file should exist, or that metadata about the software should be served by the container. If container generation software made it easy to organize and capture container content automatically, we would easily meet these goals of internal modularity and consistency, and generate containers that easily integrate with external hosts, data, and other containers.

Based on these problems, it is clear that we need direction and guidance on how organization multiple applications and data within a single container in a way that affords modularity, programattic accessibility, transparency, and consistency. This document will review the rationale and use cases for the Standard Container Integration Format (SCI-F). We will first review the goals of the architecture, followed by it's primary use case: easy integration with tools that allow for organization and comparison. For interested readers, we finish with some background on its development, and future analysis and work that is afforded by it.


## Goals
The Standard Container Integration Format (SCI-F) establishes an overall goal to make containers **consistent**, **transparent**, **parseable**, and internally **modular**.  Under these goals, we assert that a framework that produces containers, to achieve this standard, must do the following:

 - it must provide an encapsulated environment that includes packaged software and data modules, reproducible in that all dependencies are included
 - building of multiple containers must be efficient by way of allowing for re-use of common software modules
 - each software and data module must carry, minimally, a unique name and install location in the system

Each of the specific goals in context of the assertions is discussed in more detail in the following sections:

### Consistency
Given the case of two containers with the same software installed, it should be the case that the software is always found in the same location. Further, it should be the case that the data (inputs and outputs) that is used and generated at runtime is also located in a consistent manner. To achieve this goal, SCI-F defines a new root folder, `/apps`, where the container generation software should generate subfolders for each modular application installed. For example, a container with applications `foo` and `bar` would have them installed as follows:

```
/apps
   /bar
   /foo
```

Thus, if two containers both have `foo` installed, we would know to find the installation under `foo`. Data takes a similar approach. We define a new root folder, `/data`, with a similar subfolder organization:


```
/data
   /bar
   /foo
```

Thus, a container in a workflow that knows to execute the foo application would also know where to direct output, or find inputs. The details of the contents of these directories will be discussed further.

### Transparency
Arguably, when we want to know about a container's intended use, we don't care so much for the underlying operating system. We would want to subtract out this base first, and then glimpse at what remains. Given the consistent organization above, and importantly, a distinction between container base operating system (for example, software in `/bin`, `/sbin`, or even sometimes `/opt`, we can easily determine the container's additional software with a simple list command:

```
singularity exec container.img ls /apps -l

  bar
  foo
```

and we can predictably find and investigate a particular software given that we know the name:


```
singularity shell --pwd /apps/foo container.img
 
$ echo $PWD
$ /apps/foo
```

Another reason to advocate for an organization that is different from most base operating systems is because of mounting. A cluster onto which a container is mounted should be able to (in advance) know the paths that are allowed (`/apps`) and (`/data`) and not have these interfere with possibly software that already exists (and might be needed) on the cluster (e.g., `/opt`).

### Parsability
Parsability comes down to programmatic accessibility. This means that, for each software module installed, we need to be able to (easily) do the following:

 - provide an entrypoint (e.g., the current "runscript" for a Singularity container, or the Dockerfile `ENTRYPOINT` and `CMD` serve this purpose). We arguably need the different software modules within the container to each provide an entrypoint.
 - provide help. Given an entrypoint, or if a user wants to understand an installed application, it should be the case that a user can issue a command to view documentation provided for the software. For the developer, adding this functionality should be no harder than writing text in a section of a file.
 - provide metadata. A software module might have a version, a point of contact or link to further documentation, an author list, or other important metadata values that should be (also) programatically accessible.

SCI-F will accomplish these goals by way of duplicating the current singularity metadata folder, which serves to provide metadata about a container, to serve each software module installed within the container.
 
### Modularity
A container with distinct, predictable locations for software modules and data is modular. The organization of these modules under a common root ensures that each carries a unique name. Further, this structure allows for easy movement of modules between containers. If a modular carries with it information about installation and dependencies, it could easily be installed in another container. To the user, this could even look like a copy command:

```
singularity cp foo container-source.img container-dest.img
```

Given a recipe (perhaps within a module folder on the host called `foo`) we could also imagine an installation command to occur that is external (after) an original build (by way of bootstrap):


```
singularity install foo container-source.img
```

And this would allow for users to store a set of re-used bases, and then add custom changes or modules onto them. For example, I could generate images with the same modules (but diffferent bases) to test for differences based on the operating system:

```
cp ubuntu.img ubuntu-foo.img
cp centos.img centos-foo.img
singularity install foo ubuntu-foo.img
singularity install foo centos-foo.img
```

or use a common image for an operating system to generate images for different purposes all together:


```
cp ubuntu.img ubuntu-foo.img
cp ubuntu.img ubuntu-bar.img
singularity install foo ubuntu-foo.img
singularity install bar ubuntu-bar.img
```

#### What is a module?
Implied in the above organization is a decision about the level of dimensionality that we want to operate, or defining what is considered a "module." For those familiar with container technology, it is commonly the case that an entire container is considered a module.  To fully discuss this discussion, we will review the extremes.

The smallest modules, one of our extremes, would be akin to breaking containers into the tiniest pieces imaginable. We could say bytes, but this would be like saying that an electron or proton is the ideal level to understand matter. While electrons and protons, and even one level up (atoms) might be an important feature of matter, arguably we can represent a lot more consistent information by moving up one additional level to a collection of atoms, an element. In filesystem science an atom matches to a file. The problem with this level of dimensionality is that individual files aren't usually the means by which we understand software. We usually go one level up, and call a suite of software a grouping of files, in our analogy, akin to an element.

On the other extreme, we could say that an entire host (possibly running multiple containers) or even an entire cluster, is a module. This doesn't need much explanation for why the representation is not suited for developing and deploying analyses - a scientist cannot package up and share his or her entire resource in any reasonable way. Such a feat would require extensive time and money, and not be possible for the individual scientist without some extreme circumstances. 

We thus define a module between those two extremes. We can realistically define a grouping of files that are required to use software, and we can add metadata to form a complete, and programmatically understandable software package or scientific analysis. SCI-F chooses this level of dimensionality for a module because such modules can easily be put together with an operating system to get a containers.


## Integrations
The following sections discuss how such a format fits nicely to allow for integrations, including but not limit to applications and methods to generate reproducible containers, tools and workflow managers that use containers, and metrics for comparison.


### Container Bases
While containers largely provide modular, portable environments, it sometimes is the case that libraries on the host must match the container. Thus, SCI-F supports the idea of having container operating system bases, or base environments that are suited to these different needs onto which the equivalent software modules can be installed.

Under this framework, we can imagine a scenario or use case where a user is developing a container with several software modules for his or her cluster. If the container generation is managed by the cluster resource, under the hood would be provided starter bases that cater to the host needs of the user. This base image with a host operating system and possibly other libraries would be the first decision point of the container generation algorithm. The user might only need to view a list of software modules avilable given that base, and then select some subset for the image:

```
Operating System --> Library of Modules --> [choose subset] --> New Container
```

Given the same organizational rules across bases, the only filter would be with regard to which software is suited for each base, and ideally, given that software installation recipes are encouraged to be from a source base (or a package repository that is not specific to the host) most modules would work across hosts. In the case of a software module wanting to support multiple different hosts, the same rules would apply as they do now. Checks for the host architecture would come first to the installation procedure.

Likely, data containers, in that they are static (and thus not dependent on host architecture) would be consistent across operating system bases.

Under this framework, it would be possible to create an entire container by specifying an operating system, and then adding to it a set of data and software containers that are specific to the skeleton of choice. The container itself is completely reproducible because it (still) has included all dependencies. It also carries complete metadata about its additions. The landscape of organizing containers also becomes a lot easier because each module is understood as a feature.

Overall, this re-use of base containers, and sharing of software modules, creates a much more organized and less redundant environment. Operating on the level of software and data modules logically come together to form reproducible containers.


### Container Assessment
Given that a software or data module carries one or more signatures, the next logical question is about the kinds of metrics that we want and can to use to classify any given container or module. This idea has been referred to as container curation, and broadly encompasses the tasks of finding a container that serves some function, or representing containers by way of structural or functional features that can be easily compared. Akin to the discussion on levels of modularity, we will start this discussion by reviewing the different ways that we would generally use to assess containers including manual annotation, and functional assessment. We will lead into a discussion of doing this assessment based on file organization and content, a standard provided by SCI-F.


#### Manual Annotation
The obvious approach to container curation is human labeled organization, meaning that a person looks at a software package, calls it "biopython" for "biology" in "python" and then moves on. A user later might search for any of these terms and find the container. This same kind of curation might be slightly improved upon if it is done automatically based on the scientists domain of work, or a journal published in. We could even improve upon that by making associations of words in text where the container is defined or cited, and collecting enough data to have some confidence of particular domains being associated. Manual annotation might work well for small, managable projects, and automated annotation might work given a large enough source of data to learn from, but overall this metric is largely unreliable. We cannot have certainty that every single container has enough citations to make automated methods possible, or in the case of manual annotation, that there is enough manpower to maintain manual annotation.


#### Functional Assessment
The second approach to assessing containers is functionally. We can view software as a black box that performs some task, and rank/sort the software based on comparison of that performance. If two different version of a python module act exactly the same, despite subtle differences in the files (imagine the silliest case where the spacing is slightly different) they are still deemed the same thing. If we define a functional metric likes "calculates metric A from data X" and then test software across languages to do this, we can organize based on the degree to which each individual package varies from some average. This metric maps nicely to scientific disciplines (for which the goal is to produce some knowledge about the world. However, this metric is not possible to assess if we don't have a basic understanding of the container content. We could perform this test and treat containers as black boxes, but then it would be hard to know if a difference was due to some small variance in a dependency, or perhaps something about the dataset.

To play devil's advocate, let's pretend that this functional assessment serves the goal of identifying the best container for some task. We then face the challenge of requiring different domains to be able to robustly identify the metrics most relevant for this assessment, and deriving methods for measuring these metrics across new software. If you are, or have worked with scientists, you will know that this kind of agreement is hard to come by. Thus, this again is a manual bottleneck that would be hard to overtake. 

This is not to say that functional assessment should not be done or is not important. It is paramount for scientists to understand the optimal way to achieve some specific goal. However, what we would want to be able to do is a functional assessment to meet a goal, and associate different choices of software and versions to the differences in ourcomes that we see. This leads to the need for the final method for assessment, an assessment driven by container organization and content. If we have confidence in the container structure and content, we can know with complete confidence the subtle differences that we see for some functional metric can be linked back to the transparent software choices of the container. 



## Standard Container Integration Format
We now move into the standard itself, the Stanford Container Integration Format is entirely a set of rules about how a container software installs, organizes, and exposes software modules. We will start with a review of some basic background about Linux Filesystems.

### Traditional File Organization
File organization is likely to vary a bit based on the host OS, but arguably most Linux flavor operating systems can said to be similar to the [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) (FHS). For this discussion, we will disregard the inclusion of package managers, symbolic links, and custom structures, and focus on the core of FHS. We will discuss these locations in the context of how they do (or should) relate to a scientific container.

#### Do Not Touch
Arguably, the following folders should not be touched by scientific software:

 - `/boot`: boot loader, kernel files
 - `/bin`: system-wide command binaries (essential for OS)
 - `/etc`: host-wide configuration files
 - `/lib`: again, system level libraries
 - `/root`: root's home. Unless you are using Docker, putting things here leads to trouble.
 - `/sbin`: system specific binaries
 - `/sys`: system, devices, kernel features

While these locations likely have libraries and functions needed by the host to support software, it should not be the case that a scientist installs his or her software under any of these locations. It would not be easy or intuitive to find or untangle it from what is already provided by the host.

#### Variable and Working Locations
The following locations are considered working directories in that they hold variables defined at runtime, or intermediate files that are expected to be purged at some point:

 - `/run`: run time variables, should only be used for that, during running of programs.
 - `/tmp`: temporary location for users and programs to dump things.
 - `/home`: can be considered the user's working space. Singularity mounts by default, so nothing would be valued there. The same is true for..

For example, in the context of a container, it is common practice (at least in the case of Singularity) to mount the user's `/home`. Thus, if a scientist installed his or her software there, the user would not be able to see it unless this default was changed. For these reasons, it is not advisable to assume stability in putting software in these locations.

#### Connections
Connections for containers are devices and mount points. A container will arguably always need to be able to support mount points that might be necessary from its host, so it would be important for a scientific container to not put valuables in these locations.

 - `/dev`: essential devices
 - `/mnt`: temporary mounts.
 - `/srv`: is for "site specific data" served by the system. This might be a logical mount for cluster resources.


### SCI-F File Organization
The Standard Container Integration Format defines two root bases that can be known and consistently mounted across research clusters. These locations were chosen to be independent of any locations on traditional linux filesystems for the sole purpose of avoiding conflicts.

#### Apps
The base of `/apps` is where software modules will live. To allow for this, in the context of [Singularity](http://singularity.lbl.gov), we provide the following user interface for the developer to install a software module to a container. In the example below, we create a build recipe (a file named Singularity) to start with a base Docker image, ubuntu, and install app "foo" into it from Github:

```
Bootstrap: docker
From: ubuntu:latest

%appinstall foo
    git clone ...
    mkdir bin && cd foo-master
    ./configure --prefix=../bin
    make
    make install


%apphelp foo

Foo: will produce you bar.
Usage: foo [action] [options] ...
 --name/-n name your bar

%apprun foo

    /bin/bash bin/start.sh
```

In the example above, we defined an application (app) called "foo" and gave the container three sections for it, including a recipe for installation, a simple text print out to show to some user that wants help, and a runscript or entrypoint for the application, with all paths relative to the install folder. The Singularity software would do the following based on this set of instructions:

 1. Finding the section `%appinstall`, `%apphelp`, or `%apprun` is indication of an application command.
 2. The following string is parsed as the name of the application, and this folder is created, in lowercase, under `/apps` if it doesn't exist.
 3. Based on the section name (help, run), the appropriate action is taken:
   a. `%appinstall` corresponds to executing commands within the folder to install the application. These commands would previously belong in `%post`, but are now attributable to the application.
   b. `%apphelp` is written as a file called `runscript.help` in the application's base folder, where the Singularity software knows where to find it. If no help section is provided, the software simply will alert the user and show the files provided for inspection.
   c. `%apprun` is also written as a file called `runscript.exec` in the application's folder, and again looked for when the user asks to run the software. If not found, the continer should default to shelling into that location.
 4. After installation and generation of files, metadata is stored. This metadata would be stored with the container metadata, for easy parsing of all internals with one command. The user, then, could optionally provide sections `%applabels` or `%appenvironment` to define environment or labels specific to the application.
 5. Finally, as a requiremnt of installing software at bootstrap (or with another means that might be developed) checks would be done to ensure that minimal requirements such as folders not being empty are met. Bootstrap will fail if these checks do not pass.

#### Data
The base of `/data` is structured akin to apps - each installed application has a subfolder for inputs and outputs:

```
/data
   /foo
      /input
      /output
```

and software developers would be advised to make input and output paths programatically defined, and then users could easily (predictably) define and locate the data. For intermediate data, the same approach is suggested to use a temporary or working directory. As container functions and integrations are further developed, we expect this simple connection into a container for inputs and outputs specific to an application to be very useful. As for the organization and format of the data for any specific application, this is up to the application. Data can either be included with the container, mounted at runtime from the host filesystem, or connected to what can be considered a "data container."

Akin to sofware modules, overlap in data modules is not allowed. For example, let's say we have an application called foo.

 - users and developers would know that foo's data would be mounted or provided at `/data/foo`. The directory is guaranteed to exist in the container, and this addresses the issue of some clusters not being able to generate directories in the container that don't exist at runtime.
 - importing of distinct data (between bars) under that folder would be allowed, eg `/data/foo/bar1` and `/data/foo/bar2`.
 - importing of distinct data (within a bar) would also be allowed, e.g., `/data/foo/bar1/this` and `/data/foo/bar1/that`.
 - importing of overlapping content would not be allowed with a force

An application's data would be traceable to the application by way of it's identifier. Thus, if I find `/data/foo` I would expect to find the software that generated it under `/apps/foo`.


### Submodules
This discussion would not be complete without a mention for external modules or dependencies that are required by the software. For example, pip is a package manager that installs to some python base. Two equivalent python installations with different submodules are, by definition, different. There are two possible choices to take, and we leave this choice up to the generator of the container.

 - In that a python module is likely a shared dependency, or different software modules under `apps` all use python, the user could choose to install shared dependencies to a system python. In the case of conflicting versions, the user would either provide the software in entirely different containers, or install (as would be required regardless of SCI-F) different python environments per each software module.
 - The user might also choose to install python from a package resource such as anaconda, miniconda, or similar. Given this task, the anaconda (or similar) installation would be considered equivalent to any other software installed to `apps`. As the developer would do now, the folder `apps/anaconda3` would need to be installed first, and then following commands to use it directed to `/apps/anaconda3/bin/python`. If the user wanted this python to be consistently on the path, across modules, it should be added to the `%environment` section.


#### Metadata
A software or data module, in its sparsest state, is a folder of files with a name that the container points the user to. However, given easy development and definition of modules, SCI-F advocates for each application having a minimal amount of metadata:

 - in addition to a unique identifier provided by the folder name, a version number provided as a label or with running the software with `--version`
 - a content hash of it's guts (without a timestamp), which could easily be provided by Singularity at bootstrap time

The metadata about dependencies and steps to create the software would be represented in the `%appinstall`, which is by default saved with each container. Metadata about different environment variables would go into `%appenvironment`, and labels that should be accessible statically go into `%applabels`. Help for the user is provided under `%apphelp`.


## Future Work
SCI-F is exciting because it makes basic container development and usage easier. The user can immediately inspect and see the software a container provides, and how to use it. The user can install additional software, copy from one container to another, or view metadata and help documentation. The developer is provided guidance for how and where to install and configure software, but complete freedom with regard to the software itself. The minimum requirements for any package are a name for it's folder, and then optionally a runscript and help document for the user.

### Mapping of container landscape
Given separation of the software from the host, we can more easily derive features that compare software modules. These features can be used with standard unsupervised clustering to better understand how groups of software are used together. We can further apply different labels like domains and understand what modules are shared (or not shared) between scientific domains. We can find opportunity by discovering gaps, that perhaps a software module isn't used for a particular domain (and it might be).


### Artificial Intelligence (AI) Generated Containers
Given some functional goal, and given a set of containers with measurable features to achieving it, we can (either by brute force or more elegantly) programatically generate and test containers toward some metric. The lanscape of containers can easily be pruned in that the best containers for specific use cases can be easily determined automatically.
