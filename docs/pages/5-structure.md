---
layout: default
title: Structure of the Standard Container Integration Format
pdf: true
permalink: /structure
toc: true
---

# Structure of the Standard Container Integration Format

We now move into the standard itself, the Standard Container Integration Format is entirely a set of rules about how a container software installs, organizes, and exposes software modules. We will start with a review of some basic background about Linux Filesystems.

## Traditional File Organization
File organization is likely to vary a bit based on the host OS, but arguably most Linux flavor operating systems can said to be similar to the [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) (FHS). For this discussion, we will disregard the inclusion of package managers, symbolic links, and custom structures, and focus on the core of FHS. We will discuss these locations in the context of how they do (or should) relate to a scientific container. It was an assessment of this current internal standard that led to the original development of SCI-F.

### Do Not Touch
Arguably, the following folders should not be touched by scientific software:

```
/boot: boot loader, kernel files
/bin: system-wide command binaries (essential for OS)
/etc: host-wide configuration files
/lib: again, system level libraries
/root: root's home. Unless you are using Docker, putting things here leads to trouble.
/sbin: system specific binaries
/sys: system, devices, kernel features
```

While these locations likely have libraries and functions needed by the host to support software, it should not be the case that a scientist installs his or her software under any of these locations. It would not be easy or intuitive to find or untangle it from what is already provided by the host.

### Variable and Working Locations
The following locations are considered working directories in that they hold variables defined at runtime, or intermediate files that are expected to be purged at some point:

```
/run: run time variables, should only be used for that, during running of programs.
/tmp: temporary location for users and programs to dump things.
/home: can be considered the user's working space. Singularity mounts by default, so nothing would be valued there. The same is true for..
```

For example, in the context of a container, it is common practice (at least in the case of Singularity) to mount the user's /home. Thus, if a scientist installed his or her software there, the user would not be able to see it unless this default was changed. For these reasons, it is not advisable to assume stability in putting software in these locations.

### Connections
Connections for containers are devices and mount points. A container will arguably always need to be able to support mount points that might be necessary from its host, so it would be important for a scientific container to not put valuables in these locations.

```
/dev: essential devices
/mnt: temporary mounts.
/srv: is for "site specific data" served by the system. This might be a logical mount for cluster resources.
/proc connections between processes and resources and hardware information
```

## SCI-F File Organization
The Standard Container Integration Format defines two root bases that can be known and consistently mounted across research clusters. These locations were chosen to be independent of any locations on traditional linux filesystems for the sole purpose of avoiding conflicts.


### Apps
The base of /scif/apps is where software modules will live. To allow for this, in the context of Singularity, we provide the following user interface for the developer to install a software module to a container. In the example below, we create a build recipe (a file named Singularity) to start with a base Docker image, ubuntu, and install app "foo" into it from Github:

```
Bootstrap: docker
From: ubuntu:latest

%appinstall foo
    git clone ...
    mkdir bin && cd foo-master
    ./configure --prefix=../bin
    make
    make install

%applabels foo
MAINTAINER vsochat@stanford.edu

%appenv foo
FOO=BAR
export FOO


%apptest foo
/bin/bash bin/tests/run_tests.sh

%appfiles foo
README.md README.md 

%apphelp foo

Foo: will produce you bar.
Usage: foo [action] [options] ...
 --name/-n name your bar

%apprun foo

    /bin/bash bin/start.sh
```

In the example above, we defined an application (app) called "foo" and gave the container three sections for it, including a recipe for installation, a simple text print out to show to some user that wants help, and a runscript or entrypoint for the application, with all paths relative to the install folder. The Singularity software would do the following based on this set of instructions:

- Finding the section `%appinstall`, `%apphelp`, `%apprun` is indication of an application command. Not shown but also relevant are `%applabels`, `%appfiles`. and `%apptest`.
- The following string (e.g. `foo`) is parsed as the name of the application, and a folder is created, in lowercase, under `/scif/apps` if it doesn't exist.  A singularity metadata folder, `scif`, with equivalent structure to the container’s main metadata folder (`/.singularity.d`), is generated inside the application. An application thus is like a smaller image inside of it’s parent.
- A "bin" folder is automatically genereated for `foo`, and will be automatically added to `$PATH` when `foo` is being used. A "lib" folder is also generated, and is added to `$LD_LIBRARY_PATH` when `foo` is used.
- Based on the section name (e.g. "run"), the appropriate action is taken:
  - a. `%appinstall` corresponds to executing commands within the folder to install the 
application. These commands would previously belong in %post, but are now attributable 
to the application.
  - b. `%apphelp` is written as a file called runscript.help in the application's metadata folder, 
where the Singularity software knows where to find it. If no help section is provided, the 
software simply will alert the user and show the files provided for inspection.
  - c. `%apprun` is also written as a file called runscript.exec in the application's metadata 
folder, and again looked for when the user asks to run the software. If not found, the container should default to shelling into that location.
  - d. `%applabels` will write a labels.json in the application's metadata 
folder, allowing for application specific labels. 
  - d. `%appenv` will write an environment file in the application's base folder, allowing for definition of application specific environment variables. 
  - e. `%apptest` will run tests specific to the application, with present working directory assumed to be the software module’s folder


### Data
The base of `/scif/data` is structured akin to apps - each installed application has its own folder, and additionally a subfolder is created for inputs and outputs:

```
/scif/data
   /foo
      /input
      /output
```

SCI-F does not enforce or state how the container creator should use the data folders, but rather to use the organization so that a user can intutiively know that any input for app `foo` might go into `/scif/data/foo/input`, general data for `foo` might be in `/scif/data/foo`, and global data for the entire container might be in `/scif/data`. For intermediate data, the user could use a mounted data folder, or use a temporary or working directory. As container functions and integrations are further developed, we expect this simple connection into a container for inputs and outputs specific to an application to be very useful. As for the organization and format of the data for any specific application, this is up to the application. Data can either be included with the container, mounted at runtime from the host filesystem, or connected to what can be considered a "data container."

Akin to software modules, overlap in data modules is not allowed. For example, let's say we have an application called foo.

- users and developers would know that foo's data would be mounted or provided at `/scif/data/foo`. The directory is guaranteed to exist in the container, and this addresses the issue of some clusters not being able to generate directories in the container that don't exist at runtime.
- importing of datasets that follow some other specific format would be allowed, eg `/scif/data/foo/bar1` and `/sci/data/foo/bar2`.

An application's data would be traceable to the application by way of it's identifier. Thus, if I find `/scif/data/foo` I would expect to find related software under `/scif/apps/foo`.


### Environment Variables
While SCi-F is not a workflow manager, it follows naturally that the creator of a SCI-F app might want to have modules internally talk to one another. Further, the user and creator should not need to know the structural specifics of the standard, but rather how to reference them. Toward this goal, a container with SCI-F apps provides the following (automatically generated) runtime environment variables that can be used in build recipes over hard coded paths. Given the implementation in Singularity containers, these variables are prefixed appropriately:

 - `SINGULARITY_APPS`: An environment variable to point to the global apps base, `/scif/apps`
 - `SINGULARITY_DATA`: points to the global data base, `/scif/data`
 - `SINGULARITY_APPDATA`: references the app that is being run (e.g., `foo`), pointing to an app's data base (`/scif/data/foo`)
   - `SINGULARITY_APPINPUT`: references inputs (`/scif/data/foo/input`)
   - `SINGULARITY_APPOUTPUT`: references output (`/scif/data/foo/input`)

The next set of variables are defined for every app, regardless of the current running. This makes it possible to know the path to another app's data (e.g.. `bar`) while running `foo`):

 - `APPROOT_<bar>`: defined as (`/scif/apps/bar`)
 - `APPDATA_<bar>`: defined as (`/scif/data/bar`)


### Interaction
A powerful feature of container software applications is allowing for programmatic accessibility to a specific application within a container. For each of the Singularity software’s main commands, run, exec, shell, inspect and test, the same commands can be easily run for an application. 

#### Listing Applications
If I wanted to see all applications provided by a container, I could use singularity apps:

```
singularity apps container.img
bar
foo
```

#### Application Run
To run a specific application, I can use run with the --app flag:

```
singularity run --app foo container.img
RUNNING FOO
```

This ability to run an application means that the application has its own runscript, defined in the build recipe with `%apprun foo`. In the case that an application doesn’t have a runscript, the default action is taken, shelling into the container:

```
Singularity run --app bar container.img
No Singularity runscript found, executing /bin/sh
Singularity> 
```


#### Application Execution and Testing

For the commands shell and exec, in addition to the base container environment being sourced, if the user specifies a specific application, any variables specified for the application’s custom environment are also sourced.

```
     singularity shell --app foo container.img
     singularity exec --app foo container.img
```

Note that unlike a traditional shell command, we are shelling into the container with the environment and relevant context for `bar` activate. Running the command makes the assumption the user wants to interact with this software. An application with tests can be also be tested:

```
singularity test --app bar container.img
```


#### Application Inspect and Help
In the case that a user wants to inspect a particular application for a runscript, test, or labels, that is possible on the level of the application:

```
singularity inspect --app foo container.img
{
    "SINGULARITY_APP_SIZE": "1MB",
    "SINGULARITY_APP_NAME": "foo",
    "HELLOTHISIS": "foo"
}
```

The above shows the default output, the labels specific to the application foo. The user can also ask for a snippet of help text for a single app:

```
singularity help --app foo container.img
Foo: will produce you bar.
Usage: foo [action] [options] ...
 --name/-n name your bar

```


### Metadata
A software or data module, in its sparsest state, is a folder of files with a name that the container points the user to. However, given easy development and definition of modules, SCI-F advocates for each application having a minimal amount of metadata. For standardization of labels, we will follow the org.label-schema specification, for which we aim to add a simple set of labels for scientific containers

- org.label-schema.schema-version: is in reference to the schema version.
- org.label-schema.version: in addition to a unique identifier provided by the folder name, a version number provided as a label or with running the software with --version
- org.label-schema.hash-md5sum: a content hash of it's guts (without a timestamp), which could easily be provided by Singularity at bootstrap time
- org.label-schema.build-date: the date when the container was generated, or the software module was added.

The metadata about dependencies and steps to create the software would be represented in the `%appinstall`, which is by default saved with each container. Metadata about different environment variables would go into `%appenv`, and labels that should be accessible statically go into `%applabels`. Help for the user is provided under `%apphelp`.

<div>
    <a href="/SCI-F/tools.html"><button class="previous-button btn btn-primary"><i class="fa fa-chevron-left"></i> </button></a>
    <a href="/SCI-F/examples.html"><button class="next-button btn btn-primary"><i class="fa fa-chevron-right"></i> </button></a>
</div><br>
