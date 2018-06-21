# Container Service Paper

Notes and stuff for a paper I'm writing (as of summer 2018) on the XNAT Container Service.

## Reading Freesurfer reproducibility paper
[Reproducibility of neuroimaging analyses across operating systems](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4408913/)
Tristan Glatard, Lindsay B. Lewis, Rafael Ferreira da Silva, Reza Adalat, Natacha Beck, Claude Lepage, Pierre Rioux, Marc-Etienne Rousseau, Tarek Sherif, Ewa Deelman, Najmeh Khalili-Mahani, and Alan C. Evans
[doi:10.3389/fninf.2015.00012](https://dx.doi.org/10.3389%2Ffninf.2015.00012)

They cite the use of the Dice coefficient as a comparison. I don't know what that is. So I will look that up now.
[wiki](https://en.wikipedia.org/wiki/Sørensen–Dice_coefficient)

> The Sørensen–Dice index, also known by other names (see Name, below), is a statistic used for comparing the similarity of two samples.

Ok, cool. Also, later in the article...

> ...It is also commonly used in image segmentation, in particular for comparing algorithm output against reference masks in medical applications.

With a citation to this paper:
[Morphometric analysis of white matter lesions in MR images: method and validation](https://ieeexplore.ieee.org/document/363096/)
A.P. Zijdenbos; B.M. Dawant; R.A. Margolin; A.C. Palmer
[doi:10.1109/42.363096](https://doi.org/10.1109/42.363096)

Quick summary of this paper. They compared manual segmentation with segmentation from a neural network. They compared the images with essentially this Dice coefficient, which they call "a measure of similarity derived from the kappa statistic". They have an appendix in which they derive the similarity coefficient as a limit of the kappa statistic by taking the number of non-classified voxels to infinity.

Ok, back to the paper.

Here is Table 1, **Operating systems and analysis software**

|               | Cluster A                                                                     | Cluster B                                                                                 |
|:-------------:|:------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------|
|  Applications | Freesurfer 5.3.0, build 1<br>FSL 5.0.6, build 1<br>CIVET 1.1.12-UCSF, build 1 | Freesurfer 5.3.0, build 1 and 2<br>FSL 5.0.6, build 1 and 2<br>CIVET 1.1.12-UCSF, build 1 |
|  Interpreters | Python 2.4.3, bash 3.2.25, Perl 5.8.8, tcsh 6.14.00                           | Python 2.7.5, bash 4.2.47, Perl 5.18.2, tcsh 6.18.01                                      |
| glibc version | 2.5                                                                           | 2.18                                                                                      |
|       OS      | CentOS 5.10                                                                   | Fedora 20                                                                                 |
|    Hardware   | x86_64 CPUs (Intel Xeon)                                                      | x86_64 CPUs (Intel Xeon)                                                                  |

The reason I want to note this table is less about the specific values, more about the row titles and what kind of information they thought it relevant to include. (If I can editorialize for a moment, were I making this table I would have transposed it. It seems like the information in the rows here seems more appropriate to put into columns, because they are completely unrelated categories of information. But the columns here are two instances of the same thing, and I would think those should be rows. But that's just me, I guess.)

The mention of the Dice coefficient from before is explained in more detail as being used for the resting state comparisons. They processed the resting state images with FSL MELODIC with a range of different parameter options. They compared the resulting images from the two clusters, specifically the "binarized thresholded components", using the Dice coefficient.

For the Freesurfer and CIVET results, they are comparing the runs from the two clusters using the cortical thickness. The reason for this is that this measure is part of a "long pipeline" that, because there are more links in the computational chain, is more sensitive to numerical errors. When comparing the results they did not compute any metrics directly from/on the outputs (not sure why; because the surfaces are hard to work with, I guess?) but instead used a MATLAB toolbox.

> Resampled thickness files from both Freesurfer and CIVET were imported to the SurfStat MATLAB toolbox for statistical analyses.

What stats did they gather from the Freesurfer cortical thickness surfaces? In Figure 13, they show...

> surface maps of mean absolute difference, standard deviation of absolute difference, t-statistics and whole-brain random field theory (RFT) corrections for `n = 146` subjects at a significance value of `p < 0.05`, comparing the cortical thickness values extracted by Freesurfer build 1 on cluster A and cluster B.

## What to do?

I have looked around for a set of images to use, or a standard set of algorithms to test. I couldn't find anything on either end.

See, for example, [How to unit test image processing code?](https://softwareengineering.stackexchange.com/questions/166517/how-to-unit-test-image-processing-code). The question is asking how to unit test image-processing code. The only answer is essentially to come up with your own input set, run it through your process, and use those inputs/outputs as a benchmark for all future code.

I plan to use Freesurfer as one benchmark test. I could do one or two other things as well, but I don't know what they should be. On the other side, I also do not know what images to use. I could pull some off CNDA but I would need to get permission to do so. I could instead get some off TCIA, which I think makes images freely available.

## Building the test machines

The plan is to run a suite of containers on a suite of machines. The machines will be

* A VM on my iMac
* A VM on the NRG cluster
* An AWS instance
* (Possibly) A VM on my laptop

I need to have a standard build for all the test machines. We will need docker, obviously, and an XNAT with...

* The closest version to 1.7.5 we can get. (But make sure it is the same for all machines!)
* A set of plugins
    * Container service with some recent 2.0.0-SNAPSHOT version (make sure they're all the same)
    * nrg_plugin_freesurfercommon (for the FS datatype)
    * nrg_plugin_radiomics (for the radiomics datatype)
    * Possibly others if I decide on some other containers
* Containers to run
    * Freesurfer (to be finished)
    * Misha's radiomics
    * Possibly others
* Data
    * A project (called "demo")
    * Some standard set of images I haven't defined yet

### Loadout

* XNAT - commit a31b4382 (will probably be updated)
* container service - commit 0633ebaa (possibly updated, but less likely than XNAT)
* radiomics plugin - commit 77dd399
* metlab docker image - version demo.2 - hash 1145a5d30b93