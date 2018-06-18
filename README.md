# Container Service Paper

Notes and stuff for a paper I'm writing (as of summer 2018) on the XNAT Container Service.

# Reading Freesurfer reproducibility paper
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

The reason I want to note this table is less about the specific values, more about the row titles and what kind of information they thought it relevant to include.

The Dice coefficient stuff is for the resting state comparisons. They used FSL MELODIC with a range of different parameter options. They compared results for the two clusters using the "binarized thresholded components".

For the Freesurfer and CIVET results, they are using the cortical thickness. They did not compute any metrics directly on the outputs (because they are surfaces, I guess?) but instead used a MATLAB toolbox.

> Resampled thickness files from both Freesurfer and CIVET were imported to the SurfStat MATLAB toolbox for statistical analyses.

What stats did they gather from the Freesurfer cortical thickness surfaces? In Figure 13, they show...

> surface maps of mean absolute difference, standard deviation of absolute difference, t-statistics and whole-brain random field theory (RFT) corrections for `n = 146` subjects at a significance value of `p < 0.05`, comparing the cortical thickness values extracted by Freesurfer build 1 on cluster A and cluster B.

