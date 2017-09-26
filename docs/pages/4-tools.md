---
layout: default
title: Integrations and Tools
permalink: /tools
pdf: true
---

# Integrations and Tools

The following sections discuss how such a format fits nicely to allow for integrations, including but not limited to applications and methods to generate reproducible containers, supporting tools for SCI-F apps, workflow managers that use containers, and metrics for comparison.

## Container Bases
While containers largely provide modular, portable environments, it commonly is the case that libraries on the host must match the container. Thus, SCI-F is greatly enhanced by a setup that has container operating system bases, or base environments that are suited to these different needs onto which the equivalent software modules can be installed. We can imagine a scenario where a user is developing a container with several software modules for his or her cluster. Let's say that the cluster resources provides an interactive web interface to "choose your base" and then "add your software." The interface might use the following logic to guide the user's choices, and build a working container:

```
Operating System --> Library of Modules --> [choose subset] --> New Container
```

Under the hood would be provided starter bases that cater to the needs of the user. The base image with a host operating system and possibly other libraries would be the first decision point of the container generation algorithm. On the other hand, the user might not care about the base operating system, and want to first choose the software:

```
Library of Modules --> [choose module] --> Operating System  --> [add more modules] --> New Container
```

Given shared organizational rules across bases, the only filter would be with regard to which software is suited for each base, and ideally, given that software installation recipes are encouraged to be from a source base (or a package repository that is not specific to the host) most modules would work across hosts. In the case of a software module wanting to support multiple different hosts, the same rules would apply as they do now. Checks for the host architecture would come first to the installation procedure. Static data containers (under development for Singularity) in that they are not dependent on host architecture, would be consistent across operating system bases.

Under this framework, it would be possible to create an entire container by specifying an operating system, and then adding to it a set of data and software containers that are specific to the skeleton of choice. The container itself is completely reproducible because it (still) has included all dependencies. It also carries complete metadata and organization for the modules inside. The landscape of organizing containers also becomes a lot easier because each module has a specific set of content that can be easily associated. 

Overall, this re-use of base containers, and sharing of software modules, creates a much more organized and less redundant environment. Operating on the level of software and data modules logically come together to form reproducible containers.


## Container Assessment
Given that a software or data module carries one or more signatures, the next logical question is about the kinds of metrics that we want and can to use to classify any given container or module. This idea has been referred to as container curation, and broadly encompasses the tasks of finding a container that serves some function, or representing containers by way of structural or functional features that can be easily compared. Akin to the discussion on levels of modularity, we will start this discussion by reviewing the different ways that we would generally use to assess containers including manual annotation, and functional assessment. We will lead into a discussion of doing this assessment based on file organization and content, a standard provided by SCI-F.


### Manual Annotation
The obvious approach to container curation is human labeled organization, meaning that a person looks at a software package, calls it "biopython" for "biology" in "python" and then moves on. A user later might search for any of these terms and find the container. This same kind of curation might be slightly improved upon if it is done automatically based on the scientists domain of work, or a journal published in. We could even improve upon that by making associations of words in text where the container is defined or cited, and collecting enough data to have some confidence of particular domains being associated. Manual annotation might work well for small, manageable projects, and automated annotation might work given a large enough source of data to learn from, but overall this metric is largely unreliable. We cannot have certainty that every single container has enough citations to make automated methods possible, or in the case of manual annotation, that there is enough manpower to maintain manual annotation.


### Functional Assessment
The second approach to assessing containers is functionally. We can view software as a black box that performs some task, and rank/sort the software based on comparison of that performance. If two different version of a python module act exactly the same, despite subtle differences in the files (imagine the simplest case where the spacing is slightly different) they are still deemed the same thing. If we define a functional metric likes "calculates metric A from data X" and then test software across languages to do this (see the <a href="http://containers-ftw.org/apps/examples/applications/hello-world" target="_blank">hello-world</a> example), we can organize based on the degree to which each individual package varies from some average. If the output is exactly the same, we deem them perfectly equivalent. This metric maps nicely to scientific disciplines (for which the goal is to produce some knowledge about the world. However, this metric is not possible to assess if we don't have a basic understanding of the container content. In the best case that we know how to execute our assessment, without any other knowledge about container contents, it would be hard to trace back to the reason(s) for the found differences. 

Functional assessment also carries a non-trivial amount of work for the common scientist to define the metrics. Let's pretend that this functional assessment serves the goal of identifying the best container for some task. We then face the challenge of requiring different domains to be able to robustly identify the metrics most relevant for this assessment, the data, and deriving methods for measuring these metrics across new software. If you are, or have worked with scientists, you will know that this kind of agreement is hard to come by. Thus, this again is a manual bottleneck that would minimally make functional assessment a slow process. This is not to say that functional assessment should not be done or is not important. It is paramount for scientists to understand the optimal way to achieve some specific goal, sometimes regardless of the costs. 

### Modular Assessment
An enhancement to functional assessment would be having the ability to associate different choices of software and protocol to the differences in outcomes that we see. This leads to the need for the final method for assessment, an assessment driven by container organization and content. If we have confidence in the container structure and content, we can deploy a metric and then have confidence about the subtle differences that we see. For example, a single container might be used to provide ten different implementations of a sorting algorithm, the algorithms run on the same input data, and the results produced with various runtime metrics. We might programatically parse over scripts and dependencies to run each metric by way of the filesystem and build recipe, and better understand differences in performance. A "competitive container" might be defined to answer a specific scientific question, with a pre-defined metric of goodness, and scientists instructed to write the missing module to complete it. SCI-F allows the container creator to be creative in terms of how the internal modules are used.
