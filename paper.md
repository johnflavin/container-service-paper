# Introduction

[Insert introductory material about XNAT.]

One of our focuses is translating image processing algorithms from research into clinical applications.

XNAT's core functions are about getting data in, getting data out, and storing data. But once your data are in XNAT, you may want to run processing algorithms to analyze the data and generate new derived data or reports. That is why, alongside data storage, XNAT facilitates data processing and analysis. Since 200x [find out year], XNAT has shipped alongside the XNAT Pipeline Engine, which could run custom data processing "pipelines". These pipelines were written as XML documents detailing a sequence of steps to be taken for given values of input parameters. Each step tells the engine to run some executable on the command line, which has its own inputs.

The pipeline engine has been a valuable part of XNAT. Many XNAT users have also used the pipeline engine as part of analyzing their data [find citations]. But from its inception, the pipeline engine has had design problems. As an example, many of these problems can be tied to one core part of the pipeline engine: the executables required by a pipeline are not packaged as part of the pipeline.

As a direct consequence, this means that installing the pipeline XML into XNAT's pipeline engine by itself often does not allow the pipeline to run successfully. Additional executables are required for many pipelines to function. In some cases these executables are publicly available and easy to install, and the pipeline can be up and running quickly. In other cases the required executables are proprietary, not free, or difficult to install. If a group has written a pipeline around some "in-house" tool that has never been released publicly, the pipeline would be useless to outside groups.

The sharablility of pipelines is also impacted by the lack of executable packaging. If I have a pipeline on my system that I want to send to you, I cannot simply send you a repository of the pipeline XML files and expect that you will be able to install and run it. I need to tell you all of the names and versions of the dependencies you need to install. There is often a need when writing pipelines to create "setup scripts" to put the proper directories on the environment's execution path; I would need to tell you where those setup scripts are and how to edit them with the paths to the executables on your system, which may be different from the paths on my system. All this relies on the fact that I know about and can communicate how to make the pipeline work. But if, say, my environment has been customized in some way that I don't know about or have forgotten about, all your efforts to install and run the pipeline could be stymied.

The lack of packaged executables is just one example of a problem with the pipeline engine that hurt its usability to end users. But it had other problems more apparent to developers which made maintaining and improving its code base difficult. With that context, we decided that instead of using our resources to update the pipeline engine we should focus our efforts on a next-generation design for processing data in XNAT.

We designed the Container Service to meet the same needs filled by the pipeline engine while avoiding the pipeline engine's deficiencies. The fundamental need is to enable XNAT users to run external tools and scripts on their data stored in XNAT. We wanted the users' external tools to be packaged along with their dependencies; this enables easy installation, easy sharing, strong versioning, and auditing.

A natural choice to meet these needs is to build a processing infrastructure around Docker images and containers. Each processing tool is contained in a single Docker image, along with everything needed for it to run. When an XNAT user needs to process their data with this tool, a container instance is started from the image. These images are immutable, making them easy to strongly version and audit, but also easily sharable on Docker registries.

The XNAT Container Service is the infrastructure built for the purpose of processing XNAT data with Docker images. It is built as an XNAT Plugin, which is a Java jar file that can be read by XNAT to enable new functionality. The Container Service adds new REST APIs, data structures, and user interfaces to XNAT.

# Data Structures

[I assume I will list out the properties of the command and wrapper in an appendix. If so, reference that here.]

The main new data structures in the Container Service are the Command and the Command Wrapper. The Command defines the Docker image and how to run a container from it. The Command Wrapper defines how to use XNAT data to satisfy the inputs of the Command, and how to save the Command's outputs as new data in XNAT. We needed to create these data structures because we need a machine-readable way to know how to launch a Docker container from an image, but there is currently no standard way to define this. [Cite other implementations? Label-spec, boutiques, CWL]

When a Docker container is started from an image, several properties must be specified. For instance: the command to execute, environment variables, working directory, and locations at which directories and files should be "mounted" into the container from outside. The situation is analagous to sitting a person at a computer's command prompt and asking them to execute some program. What command should they run? Does the executable have inputs they need to know? Are there files they need to provide? These are all things the Container Service needs to know when launching a Docker container from an image, and this was the motivation for defining the Command. A person must write a Command to define these properties and describe a particular Docker image. They provide this Command as a JSON string to the Container Service through the user interface, the REST API, or embedded into the Docker image itself.

## Command

The Command defines all the properties known about the image, and about how to run a container from it. We will go into more detail about many of those properties below. But the most general Command properties are about the Command itself—its name and a description, for instance—and about the Docker image to be run—like its Docker image ID.

As an exmaple in this and subsequent sections, I will use the XNAT dicom to nifti command [Cite the URL]. This runs a Docker image which is the NeuroDebian Docker image with `dcm2niix` installed.

    "name": "dcm2niix",
    "description": "Executes dcm2niix in a neurodebian image",
    "label": "Run dcm2niix"
    "info-url": "https://github.com/rordenlab/dcm2niix",
    "version": "1.4",
    "schema-version": "1.0",
    "type": "docker",
    "image": "xnat/dcm2niix:1.4"

## Command-line

The core property of the Command is the `command-line` string, used to construct the command to execute inside the container. The example from the `xnat/dcm2niix` Command is:

    "command-line": "dcm2niix [BIDS] [OTHER_OPTIONS] -o /output /input"

Some of the strings, `[BIDS]` and `[OTHER_OPTIONS]` are template strings. These will be resolved with values at the time a container is to be launched. The file paths `/output` and `/input` are file paths which will be mounted from the outside at runtime.

## Inputs

The template strings in the command line are resolved by the values given at runtime to inputs. This is the definition of the input that gives the value to the `[OTHER_OPTIONS]` template above:

    {
        "name": "other-options",
        "description": "Other command-line flags to pass to dcm2niix",
        "type": "string",
        "required": false,
        "replacement-key": "[OTHER_OPTIONS]"
    }

At runtime, the value for this input must be provided to the Container Service. If the value provided is `"other-options": "-z"`, the Container Service will find all the template strings containing the `replacement-key` value `[OTHER_OPTIONS]` and replace them with the value `-z`.

Inputs can be replaced differently in the command line string than in other strings if they specify a `command-line-flag` property and a `command-line-separator` property. Boolean inputs can use the `true-value` and `false-value` properties to control how the `true` and `false` values are mapped to strings. The `bids` input shows both of these features.

    {
        "name": "bids",
        "description": "Create BIDS metadata file",
        "type": "boolean",
        "required": false,
        "default-value": false,
        "replacement-key": "[BIDS]",
        "command-line-flag": "-b",
        "true-value": "y",
        "false-value": "n"
    }

If the input is not given a value at runtime, it takes its default value of `false`. The template string `[BIDS]` in the command line will be replaced with the resolved value `-b n` using the `command-line-flag` and `false-value` properties. If the runtime value is `true`, then the template string `[BIDS]` will be replaced by `-b y`.

## Mounts

A mount is a file path created in the container which is connected to a directory outside the container. This is the mechanism for bringing XNAT files into the container, and for retrieving the container's output files when it is complete. Any files written by the container to a path that is not mounted to the outside will be lost when the container is complete. The most important property of the mount is its `path`, which is the path inside the container where the mount will be created.

The two mount paths in the `xnat/dcm2niix` command line string above are `/input` and `/output`, defined in the command as the mounts `dicom-in` and `nifti-out`, respectively.

    [
        {
            "name": "dicom-in",
            "writable": "false",
            "path": "/input"
        },
        {
            "name": "nifti-out",
            "writable": "true",
            "path": "/output"
        }
    ]

The author of the Command sets the path at which the mount will appear within the container. The path outside the container which will be mounted is controlled by the Container Service. If the mount is intended for input data, properties in the Command Wrapper's inputs can be set in such a way that the Container Service will mount data from particular locations within XNAT's archive directly into the container. These mounts are not writable. If no particular input data is specified for a mount, the Container Service assumes the mount is intended to store output data; it creates a new writable directory to hold output files.

## Outputs

If the container produces files which will be handled by importing them back into XNAT, an output object must be defined. An output must point to a mount; the files within the mount may then be uploaded. In the simplest case, all the files in the mount are taken as-is. In more complex cases the output can be restricted to a certain sub-path within the mount, or to files with names matching a given pattern.

The `xnat/dcm2niix` Command uses the simple form of the output, which merely points to a mount.

    {
        "name": "nifti",
        "description": "The nifti files",
        "mount": "nifti-out",
        "required": "true"
    }

## Additional Properties

There are other properties that can be set on the Command if needed, such as environment variables, open ports, and working directory. These can be seen in the detailed Command specification in [link to Appendix].

## Command Wrapper

The Command Wrapper is the section in which a definition can be given for how to use XNAT data to satisfy the Command's inputs and provide files to its mounts, and how to handle its outputs. All the properties of the Command detailed so far are platform-agnostic in that they could be adopted by any platform as a way to describe the execution of Docker containers from images. The Command Wrapper is platform-specific, and is only applicable to the XNAT Container Service.

The Command Wrapper includes the same properties as many other things in the Command: `name`, `description`, and `label`. It has a property called `contexts`, which takes a list of XNAT data types for which the wrapper should appear in the Container Service user interface. And it has inputs and output handlers, which interact with the Command's inputs and outputs.

The Command for the `xnat/dcm2niix` image has a Wrapper intended to take an XNAT scan. Its first few properties are:

    "name": "dcm2niix-scan",
    "description": "Run dcm2niix on a Scan",
    "contexts": ["xnat:imageScanData"]

## Wrapper Inputs

The Command Wrapper has two kinds of inputs: external and derived. The external inputs are those for which a value must be provided at runtime. Often these inputs represent XNAT objects like projects, sessions, or scans, and the input value is a unique identifier for a particular object. The derived inputs represent other XNAT objects, or properties of those objects, that can be defined by their relationship to an object that has already been defined by a Wrapper input. So, for instance, if we have a external input with type `Session`, we could make derived inputs for that session's scans or resources, or we could have derived inputs which take values from properties of the session, like its ID or label.

The wrapper inputs can be used to provide values to the command inputs by giving a Command input as the value for the property `provides-value-for-command-input`. And if the object described by the Command Wrapper input has files in XNAT, like a session or a resource, for instance, the input can set the `provides-files-for-command-mount` property to the name of a mount. This will cause the Container Service to mount the object's files from the XNAT archive into the container.

The `dcm2niix-scan` wrapper has one external input for an XNAT scan and one derived input for that scan's DICOM resource.

    "external-inputs": [
        {
            "name": "scan",
            "description": "Input scan",
            "type": "Scan",
            "required": true,
            "matcher": "'DICOM' in @.resources[*].label"
        }
    ],
    "derived-inputs": [
        {
            "name": "scan-dicoms",
            "description": "The dicom resource on the scan",
            "type": "Resource",
            "derived-from-wrapper-input": "scan",
            "provides-files-for-command-mount": "dicom-in",
            "matcher": "@.label == 'DICOM'"
        }
    ]

## Output Handlers

stuff

# Benchmarking / Validation

Run pyradiomics, talk about results.

# Use Cases

## Retrospective Analysis

If you have thousands of patient records with data that you want to process...

Talk about batch launching.

What should I say about this? We can do batch launching but it ain't great. In particular, we don't have a scheduling layer, so I wouldn't say you should launch them all at once.

## Real-Time Analysis

Talk about event service integration. Talk about TIP workflow & pushing data back through PACS?

# Discussion
