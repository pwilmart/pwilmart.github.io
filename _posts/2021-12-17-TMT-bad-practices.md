---
layout: post
title: "TMT bad practices"
date: 2021-12-17
---

# Many established practices in TMT analyses are bad

## Part 1

This will be a "shoot the messenger" exercise. I have strong opinions on many things, including the science that I do. I try really hard to shape my science views based on facts, direct observations, and extrapolation of well-established concepts. I have tested or thought deeply about every point listed below.

I hope others will consider the possibility that they may be doing things less well than they could. Many of the issues below do not invalidate research findings (no paper is 100% right or 100% wrong). The gains for individual issues may not be earth shaking by themselves. These issues tend to be additive (maybe multiplicative?), so the **overall gains** may be larger than you would predict.

## MS2 versus MS3 for reporter ions
If you have access to instruments that can do the SPS MS3 methods, you should stop doing MS2-based reporter ions. Information loss of reporter ion signals (it's not really “ratio” compression - see https://pwilmart.github.io/blog/2020/01/05/TMT-ratio-distortions) comes from additive background signals in all the reporter ion channels. This background can be similar in all channels and have the appearance of a compression effect. This is often incorrectly considered a benign effect: measured fold changes will be smaller than actual fold changes, but one can still distinguish differentially abundant proteins from unchanged proteins. Some smaller fold-change proteins may get lost in the unchanged background (false negatives), but you can still get the bigger biology picture.

While some of that may be true, the flip side is ignored. What if the background is not flat? The background is not a true non-specific background (what mistakenly led to calling the distortion a mean-bias effect). The background is something picked up in the precursor isolation window. It depends on retention time and m/z. What happens when the background might have structure (i.e., not be flat)? You can get false positives from the unchanged proteins (see https://github.com/pwilmart/PXD013277_E-coli_spike-ins_MS2-TMT). Are you sure that you will still get the bigger biology picture correct?

My gut feeling is that MS2-based reporter ions are not good quality measurements. It may work in some cases, but so does spectral counting. “Quantifying” more proteins poorly is not as good a choice as some try to make it sound.

## Signal-to-noise ratios
A key to measurement science is knowing what to measure. An unnamed vendor, based on feedback from an unnamed prominent lab, mistakenly measures reporter ion signal-to-noise ratios instead of reporter ion intensities (peak heights as proxies for peak areas). S/N ratios are not physically meaningful and are not actual units of measurement. They are unit-less ratios. S/N ratio is a qualitative measure of signal quality (mostly in electrical signal contexts).

Ion traps do not measure ion current like continuous beam instruments would do. They limit the number of ions in the trap based on a desired target value to reduce space charging effects. How **quickly** the trap gets to the target is how the ion current is estimated. I use ion current “estimate” because that is what it is. There are many ways for the scaled “intensities” to be inaccurate. However all the behind-the-scenes engineering works, we know from experience that the intensities are proportional to analyte concentrations. That is why we can make standard curves and do quantitative assays using ion traps. I recall some argument that S/N ratios are more correlated with the number of ions in the trap. However, we already know that because of the automatic gain control target. This is not a compelling argument.

A more fundamental issue is what do you do with a noise estimate when you are measuring something that is signal+noise? The answer is not to take a ratio. That does not give you the estimate of the signal. You have to subtract the noise from the signal+noise to get the signal estimate. This is already done on Orbitraps. You may argue that S/N ratio can be used to define a weak signal cutoff to improve data quality. Yes, but you can also do that with the magnitude of the reporter ion peak height directly.

There is nothing that is gained from using S/N ratios, and much that is lost. Summed reporter ion intensity for proteins is correlated with MS1 feature intensities and is a reasonable proxy for relative protein abundance. S/N ratios are a compressed numerical scale and are not a very good proxy for relative protein abundance. I have studied PTMs in lens proteins since I started in proteomics in 2003. There is a lot known about the human lens proteome. A handful of special crystallin proteins make up over 90% of the total wet weight of lens proteins. If you compute the fraction of crystallins of the total using MS1 features, you get over 90% crystallins. If you use summed reporter ion signals, you get 90% crystallins. If you use spectral counting you get about 30% crystallins. We know that spectral counting under-estimates high abundance proteins and over-estimates low abundance proteins, so the 30% value is not surprising. If you used summed S/N ratios of reporter ions for lens, you get about 60% crystallins. S/N ratios are not as distorted as spectral counts, but they are distorted. Why use a “measurement” value that does not give you the correct answer?

## Ratio and log transformations
Fold changes are an established way to **summarize** quantitative differences in biological systems. The key word in that sentence is “summarize”. Fold changes (and the related log2 FC) are something you compute **after** you do the analysis. They are not something you do the analysis with.

Ratios are not proper measurement scales. They are dimensionless quantities, and things we measure have units. They are non-linear, compressed mathematical spaces. Half of any ratio space goes from zero to one. The other half goes from one to infinity. That asymmetry can be remedied by making a logarithmic transformation (base 2 is the most common), but that further compresses the mathematical space. This does not fix "the problem". We do not need all proteins in a proteome to have relative abundance be normally distributed for statistical testing. We are testing one protein at a time. We need to have quantities for a given protein be normally distributed for things like t-tests. We usually do not have enough replicates to test that anyway.

Natural measurement scales (what the measuring device produces) are more intuitive and make quality control assessment much easier. In TMT experiments, the natural scale is reporter ion intensity. Doing as much of an analysis in the natural scale as possible facilitates understanding and interpretation. **I cannot overstate how important this is.**

## Isotopic purity corrections
I do not apply reagent purity corrects in my TMT analyses. Here are some reasons why. Most correction algorithms apply corrections at the individual scan level. The reporter ion signals from individual scans are most often poorer quality than reporter ion values for peptides or proteins. The uncertainties in the reporter ion values in individual scans propagate into errors in the corrected values. This is most problematic for missing values, which can be reduced after aggregation. If one wanted to try applying corrections to peptides or protein values, that has to be carefully coordinated with any normalization adjustments. The order of operations matters here. Corrections need to be done before any global channel scalings are done.

Most of the work in this area was done before high-resolution reporter scans were used to resolve the N- and C-series TMT tags. The two series are mostly independent in terms of isotopic purity cross talk. The “-1” m/z values for N-series tags need some additional thinking. You probably get doublets from C12 and N14 impurities in the C13 and N15 isotopes (around 99% purity). One of those doublets may not match any lighter tag m/z slots. Ironically, the 6-plex reagents are alternating N- and C- forms. If high-resolution scans are used for the reporter ions, you do not really need to worry about correction factors for 6-plex kits.

Isotopic impurity cross talk limits the dynamic range of TMT measurements. I think about 10-fold changes are the best you can get (when adjacent channel intensities are present). That is workable for most quantitative study designs. It has to be kept in mind for some specific questions. TMT experiments are not very good ways to see if something is **not** there.

## How to summarize PSM measurements into protein measurements
It should not come as a surprise that signal processing (averaging, summing) as one aggregates PSM scans to peptides, and peptides to proteins improves data quality. See https://pwilmart.github.io/TMT_analysis_examples/MAN1353_peptides_proteins.html for an example with TMT data. Aggregation dramatically reduces measurement dimensionality (some see this as beneficial), but, as with any reduction in data, there may be loss of information. That is another complicated debate that we will not revisit here.

There are many ways to aggregate data. Summation of intensities are commonly used in MS1 label-free analyses (MaxQuant). Proteome Discoverer sums TMT PSM values into protein totals. I think (I might be wrong) Skyline sums transition peak areas into peptide totals. In some ways, summation is a weighted averaging because larger values contribute more to the total.

An alternative used in MS1 label-free work is the top 3 method from Silva, et al. (https://www.sciencedirect.com/science/article/pii/S1535947620315127). The idea is that a protein at a given concentration, when digested by trypsin, liberates peptides at the same concentration. Many factors affect the mass spec signal of individual ions (peptides in given charge states). They can stick to surfaces, they may not bind to columns, they may bind too tightly to columns, they may not become ions as droplets desolvate (the ionization process depends on competition for H+). If we are lucky, we can get m/z signals approaching the level of the parent protein. Most of the m/z signals (the peptides) will be suppressed by unknown, arbitrary factors and have reduced signals. The top 3 method averages the top 3 most intense signals to estimate the maximum peptide signal to use as a proxy for the parent protein signal.

The difference between top 3 and summing up everything is a little like the difference between molar concentration and weight percentage concentration. Both have been shown to correlate with protein relative abundance.

Median value summarizations are popular in statistics. They are really alternatives to averaging as ways to determine the center of distributions. They are not always physically meaningful ways to summarize actual measurements. If many people measured the length of a table independently, the measurements might have a Gaussian distribution around the true value, and the center of the distribution might be a good estimate of the table length. The median of a collection of PSM values for a protein is a different situation. The median of PSM signals does not have any physical meaning. It is not related to molar concentration like the maximum signal might be. It is not like a relative weight percentage in the way that summing everything seems to be. Because the suppression of individual PSMs is arbitrary, the median PSM signal becomes an **arbitrary** protein value. Whether there are mathematically desirable characteristics of median summarization or not is secondary to loss of interpretability of the protein values.

I use summation of reporter ion intensities for all unique peptides to estimate relative protein abundances. Defining shred and unique peptides is not trivial and I have some blog posts about that: https://pwilmart.github.io/blog/2019/09/21/shotgun-quantification and https://pwilmart.github.io/blog/2020/09/19/shotgun-quantification-part2. These reporter ion protein intensities are pretty nicely correlated with MS1 feature intensities and provide some relative protein abundance information that is very important for biological interpretation. Any protein summary from MS2-based data will have variability from pseudo random sampling in the selection of MS2 scans, but it is still useful information (if taken with a grain of salt).

## Part 2 Preview

I have many other more minor issues, but I will save those for another day.

- FASTA files with significant peptide redundancy
- Use of poor search engines and/or poor settings
- Multi-TMT-plex experimental designs and use of bridge/reference channels
- Understanding of all the necessary normalizations
- Protein inference and definition of usable peptides
- Use of replicates, experimental design, and statistical testing
- Visualizations for quality control and sanity checks
- Ability to document an analysis 

---
Happy holidays everyone! <br> Phil Wilmarth <br> December 17, 2021
