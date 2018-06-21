# Introduction

About XNAT.

XNAT's core functions are about getting data in, getting data out, and storing data. But it also provides functions for generating new ancilary data from your existing data. [ugh this is bad]

But alongside data storage, XNAT facilitates data processing and analysis. Since 200x, XNAT has shipped alongside the XNAT Pipeline Engine, which could run custom data processing "pipelines". These pipelines were XML documents detailing a sequence of steps to be taken for given values of input parameters. Each step tells the engine to run some executable on the command line, which has its own inputs.

The pipeline engine has been a valuable part of XNAT, and contributed to many results (citations needed). But from its inception it has had design problems.

## The executables required by the pipeline are not packaged as part of the pipeline.
This means that installing the pipeline XML into XNAT's pipeline engine by itself often does not allow the pipeline to run successfully. Additional executables are required for many pipelines to function. In some cases these executables are publicly available and easy to install, but in other cases they are proprietary, not freely available, or difficult to install.

As a corollary, this difficulty in ensuring the availability of a pipeline's required executables means that pipelines are not "portable". Installing and successfully running a pipeline on one machine or in one environment does not mean it can be easily transferred to another.

# Implementation


# Benchmarking / Validation


# Discussion
