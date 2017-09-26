---
layout: default
title: Community Development
---

# Appendix

## Discussion
This discussion would not be complete without a mention for external modules or dependencies that are required by the software. For example, pip is a package manager that installs to some python base. Two equivalent python installations with different submodules are, by definition, different. There are two possible choices to take, and we leave this choice up to the generator of the container.

 - In that a python module is likely a shared dependency, or different software modules under `apps` all use python, the user could choose to install shared dependencies to a system python. In the case of conflicting versions, the user would either provide the software in entirely different containers, or install (as would be required regardless of SCI-F) different python environments per each software module.
- The user might also choose to install python from a package resource such as anaconda, miniconda, or similar. Given this task, the anaconda (or similar) installation would be considered equivalent to any other software installed to apps. As the developer would do now, the folder /scif/apps/anaconda3 would need to be installed first, and then following commands to use it directed to `/scif/apps/anaconda3/bin/python`. If the user wanted this python to be consistently on the path, across modules, it should be added to the `%environment` section.

In practice, we have found that global installs tend to be larger, well maintained libraries (e.g., libraries installed with `apt-get` or package managers like `pip`) and having them represented in the `%post` section, to be shared among apps, helps with any kind of analysis that wants to separate what might be considered the general container "base" against the different custom software installed.

We do not enforce using SCI-F for Singularity images or any other container. It's creation and discussion is implemented and provided to only help scientists more easily create reproducible, transparent containers.


## Conclusion
The Standard Container Integration Format is advantageous in that the container creator can embed his or her work with implied metadata about software and container contents. SCI-F also makes it easier to package different run scripts with the container, and expose them easily to the user. However, this does not mean that the standard approach of using a container as a general toolbox and distributing it with a series of external callers is bad or wrong. The choice to use (or not use) SCI-F apps is largely dependent on the goals of the creator, and the intended users.


## List of (possibly) related standards,formats and initiatives

 - FHS: https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard
 - OCI: https://github.com/opencontainers/image-spec/blob/master/spec.md
 - CWL: http://commonwl.org
 - FAIR: https://www.force11.org/group/fairgroup/fairprinciples
 - (Opt.) OpenStand: https://open-stand.org
 - https://reproducible-builds.org/ 
 - DASPOS https://daspos.crc.nd.edu
 - TANGO http://tango-project.eu/

## Additional reading material

 - https://indico.esrf.fr/indico/event/6/session/2/contribution/3/material/slides/
 - https://indico.cern.ch/event/567550/contributions/2656689/
 - https://github.com/TANGO-Project/alde#application-lifecycle-deployment-engine-alde
 - https://figshare.com/articles/Orchestration_and_Workflows_in_eScience_Problems_Standards_and_Solutions/4746931

