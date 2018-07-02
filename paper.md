# Introduction

About XNAT.

One of our focuses is translating image processing algorithms from research into clinical applications.

XNAT's core functions are about getting data in, getting data out, and storing data. But it also provides functions for generating new ancilary data from your existing data. [ugh this is bad]

But alongside data storage, XNAT facilitates data processing and analysis. Since 200x, XNAT has shipped alongside the XNAT Pipeline Engine, which could run custom data processing "pipelines". These pipelines were XML documents detailing a sequence of steps to be taken for given values of input parameters. Each step tells the engine to run some executable on the command line, which has its own inputs.

The pipeline engine has been a valuable part of XNAT, and contributed to many results (citations needed). But from its inception it has had design problems. As an example, many of these problems can be tied to one core part of the pipeline engine: the executables required by a pipeline are not packaged as part of the pipeline.

As a direct consequence, this means that installing the pipeline XML into XNAT's pipeline engine by itself often does not allow the pipeline to run successfully. Additional executables are required for many pipelines to function. In some cases these executables are publicly available and easy to install, and the pipeline can be up and running quickly. In other cases the required executables are proprietary, not free, or difficult to install. If a group has written a pipeline around some "in-house" tool that has never been released publicly, the pipeline would be useless to outside groups.

The sharablility of pipelines is also impacted by the lack of executable packaging. If I have a pipeline on my system that I want to send to you, I cannot simply send you a repository of the pipeline XML files and expect that you will be able to install and run it. I need to tell you all of the names and versions of the dependencies you need to install. There is often a need when writing pipelines to create "setup scripts" to put the proper directories on the environment's execution path; I would need to tell you where those setup scripts are and how to edit them with the paths to the executables on your system, which may be different from the paths on my system. All this relies on the fact that I know about and can communicate how to make the pipeline work. But if, say, my environment has been customized in some way that I don't know about or have forgotten about, all your efforts to install and run the pipeline could be stymied.

The lack of packaged executables is just one example of a problem with the pipeline engine that hurt its usability to end users. But it had other problems more apparent to developers which made maintaining and improving its code base difficult. With that context, we decided that instead of using our resources to update the pipeline engine we should focus our efforts on a next-generation design for processing data in XNAT.

# Design

We designed the Container Service to meet the same needs filled by the pipeline engine while avoiding the pipeline engine's deficiencies.

# Implementation

The Container Service is an XNAT plugin.

# Benchmarking / Validation


# Discussion
