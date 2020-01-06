---
layout: post
title: "TMT Ratio Distortions"
date: 2020-01-05
---

# Understanding MS2 TMT ratio inaccuracies

## Backgound

We have been aware that isobaric labeling data from MS2 scans has had some accuracy issues for more than a decade. The path to understanding this topic has been a slow one and I do not think we are all of the way there yet.

> Ow, S.Y., Salim, M., Noirel, J., Evans, C., Rehman, I. and Wright, P.C., 2009. iTRAQ underestimation in simple and complex mixtures:“the good, the bad and the ugly”. Journal of proteome research, 8(11), pp.5347-5355.

> Ting, L., Rad, R., Gygi, S.P. and Haas, W., 2011. MS3 eliminates ratio distortion in isobaric multiplexed quantitative proteomics. Nature methods, 8(11), p.937.

> Karp, N.A., Huber, W., Sadowski, P.G., Charles, P.D., Hester, S.V. and Lilley, K.S., 2010. Addressing accuracy and precision issues in iTRAQ quantitation. Molecular & Cellular Proteomics, 9(9), pp.1885-1897.

The first ideas were that might be co-eluting peptides that have different expression patterns getting mixed together. As we switched from lower resolution instruments to higher resolution ones, it became more feasible to filter for precursor ion isolation purity. That can also be addressed by larger-scale peptide separations where co-eluting features would be less likely. The bottom line was that co-elution did not seem to be the issue.

In much the same way that using both parent ion mass filters and fragment ion mass filters improves selected reaction monitoring, using an extra round of isolation before generating the fragment ions should reduce background noise levels. The new Tribrid instruments from Thermo enabled such acquisition modes that improved reporter ion accuracy at the exspense of some duty factor and signal strength. There is also the literal expense of the instruments to contend with.

> McAlister, G.C., Nusinow, D.P., Jedrychowski, M.P., Wühr, M., Huttlin, E.L., Erickson, B.K., Rad, R., Haas, W. and Gygi, S.P., 2014. MultiNotch MS3 enables accurate, sensitive, and multiplexed detection of differential expression across cancer cell line proteomes. Analytical chemistry, 86(14), pp.7150-7158.

> Senko, M.W., Remes, P.M., Canterbury, J.D., Mathur, R., Song, Q., Eliuk, S.M., Mullen, C., Earley, L., Hardman, M., Blethrow, J.D. and Bui, H., 2013. Novel parallelized quadrupole/linear ion trap/Orbitrap tribrid mass spectrometer improving proteome coverage and peptide identification rates. Analytical chemistry, 85(24), pp.11710-11714.

## Current state

I see researchers using both methods (MS2 reporter ions or SPS MS3 reporter ions) like they are equally good. If you do not have access to a Tribrid Orbitrap (Fusion), you might have a Q-Exactive (QE) handy. You might think that it really does not matter where the reporter ions are measured or that you might get deeper proteome profiling on the somewhat faster scanning QE.

Scan speeds and sequencing depth are moving targets. For FT instruments, cycle times depend on resolution. How long it takes to achieve certain resolutions can also depend on the particular instrument. Increased sequencing speed is usually improved with each new Orbitrap model. The Tribrid and QE platforms are not completely in sync in terms of new model release dates, so similar age instruments may not have similar scan rates. We have a first generation Fusion and a QE-HF instrument. Our QE is faster than SPS MS3 on the Fusion. We get more sequencing attempts per unit time on our QE compared to our Fusion for TMT experiments. How that translates into sensitivity improvements depend on many factors: the sample, the sample load, the LC elements, the LC separation, instrument cleanliness, etc. Newer tribrids are faster and more sensitive than our 6-year old Fusion.

Measurement accuracy is also a non-trivial question to address. We seldom have ground truth experiments that fully mimic real world samples. Many of the spike in style experiments are a little too simple. Those have already been used to show that MS2 data is less accurate than SPS MS3 data. Expression difference accuracy is not the whole enchilada, though. We might be satisfied to know which proteins have changed in expression, even if that measured change is not 100% accurate. Now the question has morphed from "is MS2 TMT data from a QE perfectly accurate" to "is MS2 TMT data from a QE usable"?

Maybe the only way to really know is to run the same samples on a Tribrid instruments using SPS MS3 and on a QE instrument using MS2 with similar LC systems and separations. After taking into account different dataset sizes, etc., some sort of head-to-head comparison could be attempted. Then (maybe) you have answered the question, but only for one specific biological experiment. Is that experiment generalizable to others?

> I am working on a biological experiment where we did run the same TMT-labeled samples on both the Fusion and the QE. I am about done and may be able to have something up at Github (https://github.com/pwilmart/TMT_analysis_examples) in a few days. These comparisons are never as apples-to-apples as you would think that they should be.

## Can we think our way out of this?

What do we know? Maybe we are fooled by the high resolution precursor scans. We see the narrow peaks and ignore all of the space in between the peaks. While sharp peaks can be quite tall relative to some noise baseline, a very wide (but not very tall) rectangle can have a large area. The isolation quadrupole is still low resolution. Transmission efficiency for narrower isolation widths has improved, so that we can maybe use 0.5 Da instead of 2.0 Da now. That would be 4 times less noise. However, the isolation width is still **enormous** compared to 4 or 5 peaks of 0.01 Da or less (a typical isotopic distribution). The integrated noise can really add up.

What I think we have going on for MS2 reporter ions is non-specific (but TMT labeled) background noise being isolated with our peptide of interest. That background contributes a more-or-less constant signal to all of the reporter ion channels. Even if we have only a single precursor being isolated, when the precursor abundance is low, the noise level might be significant.

We can mock that up for a few reporter ion abundances (1000, 10000, and 100000) with a constant background level (500) for some fold changes (1.0, 1.25, 1.5, 2, 3, 5, and 10). We can see how a constant background added to each state affects the expression ratio. We will be using the US nomenclature of "`,`" for thousands separator, and "`.`" for the decimal point.

### Ratio distortions with State 2 up regulated compared to State 1

**State 1 at 1,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected R|R with 500 Bgd
-------|-----------|-------|----------|--------------
1,000|1.00|1,000|1.00|1.00
1,000|1.25|1,250|1.25|1.17
1,000|1.50|1,500|1.50|1.33
1,000|2.00|2,000|3.00|1.67
1,000|3.00|3,000|3.00|2.33
1,000|5.00|5,000|5.00|3.67
1,000|10.00|10,000|10.00|7.00

**State 1 at 10,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected R|R with 500 Bgd
-------|-----------|-------|----------|--------------
10,000|1.00|10,000|1.00|1.00
10,000|1.25|12,500|1.25|1.24
10,000|1.50|15,000|1.50|1.48
10,000|2.00|20,000|3.00|1.95
10,000|3.00|30,000|3.00|2.90
10,000|5.00|50,000|5.00|4.81
10,000|10.00|100,000|10.00|9.57

**State 1 at 100,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected R|R with 500 Bgd
-------|-----------|-------|----------|--------------
100,000|1.00|100,000|1.00|1.00
100,000|1.25|125,000|1.25|1.25
100,000|1.50|150,000|1.50|1.50
100,000|2.00|200,000|3.00|2.00
100,000|3.00|300,000|3.00|2.99
100,000|5.00|500,000|5.00|4.98
100,000|10.00|1,000,000|10.00|9.96

As the signal gets larger compared to the noise, the ratios get closer to the expected values. The above examples had **State 2** being over-expressed. What about the opposite case where we have similar fold changes but **State 2** is under expressed? We will keep the same 1,000 signal in **State 1**, but smaller signals in **State 2**. We will invert the ratio so we can keep the same expected values (to make comparing things easier).

### Ratio distortions with State 2 down regulated compared to State 1

**State 1 at 1,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected 1/R|1/R with 500 Bgd
-------|-----------|-------|----------|--------------
1,000|1.00|1,000|1.00|1.00
1,000|1.25|800|1.25|1.15
1,000|1.50|667|1.50|1.29
1,000|2.00|500|3.00|1.50
1,000|3.00|333|3.00|1.80
1,000|5.00|200|5.00|2.14
1,000|10.00|100|10.00|2.50

**State 1 at 10,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected 1/R|1/R with 500 Bgd
-------|-----------|-------|----------|--------------
10,000|1.00|10,000|1.00|1.00
10,000|1.25|8,000|1.25|1.24
10,000|1.50|6,667|1.50|1.47
10,000|2.00|5,000|3.00|1.91
10,000|3.00|3,333|3.00|2.74
10,000|5.00|2,000|5.00|4.20
10,000|10.00|1,000|10.00|7.00

**State 1 at 100,000. Background of 500 added to both states.**

State 1|Fold Change|State 2|Expected 1/R|1/R with 500 Bgd
-------|-----------|-------|----------|--------------
100,000|1.00|100,000|1.00|1.00
100,000|1.25|80,000|1.25|1.25
100,000|1.50|66,667|1.50|1.50
100,000|2.00|50,000|3.00|1.99
100,000|3.00|33,333|3.00|2.97
100,000|5.00|20,000|5.00|4.90
100,000|10.00|10,000|10.00|9.57

Okay, this was a little deceptive. The case where we start with **State 1** at 1,000 and then decrease things for **State 2** just increases the relative amount of noise. That distorts the ratios even more. We have some background noise level. The closer either the numerator or the denominator gets to the noise value, the more the ratio will be distorted. What we are talking about here are ratios for individual PSMs. If a feature is more abundant, the noise is less of a factor and the ratio is closer to its true value. If a feature is close to the noise level, the noise can severely distort the measured ratio from the true value.

#### In any measurement of a signal, the way to deal with a background level is to SUBTRACT the background. Doing something else, like taking a signal-to-noise ratio, is not valid. It compresses the measurement scale, and does not remove the noise. Proteome Discoverer uses S/N ratios instead of reporter ion intensities as the default. I do not recommend using S/N ratios (ever).  

If you are measuring things at the PSM level, you have to be very aware of this potential for distortion. If you are aggregating PSMs to higher levels like proteins, then the relative effects of low abundance PSMs may be mitigated. It depends on lots of other factors: How many PSMs were aggregated into the protein value? Were some of the PSMs at moderate to higher abundance? Was the aggregation a sum of reporter ion signals?

If you are determined to make your work harder and your results less robust by working with expression ratios rather than intensities, the distortion from a constant noise level will operate differently. Summing PSM reporter ion signals into protein totals reduces the distortion from low abundance features because more abundant (and less distorted ratios) contribute more to the totals. Methods like forming ratios of each PSM and then taking the median of the ratios has the effect of equally weighting distorted ratios (low abundance PSMs) and more accurate ratios. It is also worth remembering that many proteins in most experiments have small numbers of associated PSMs. A median is not a robust estimator for small numbers of data points.

## Channel cross-talk is a similar kind of problem

Each isobaric tag has isotopic effects. There is some signal at +1 Da positions from carbon 13, and some signal at -1 Da from the purity of the heavy atoms. So each channel has its signal and some background signals from adjacent channels. When a channel's signal is relatively small and the cross-talk background from adjacent channels is comparable, you will get distorted measurements. Corrections can be attempted if the cross-talk levels are known (i.e. manufacturer spec sheets). The problem changes if the reporter ion resolution is high (like we use for 10-, 11-, or 16-plex reagents). Then the N- and C-series tags are somewhat decoupled as they tend to interfere just within series. The -1 Da mass shift between C13 and C12 is also different than the mass shift between N15 and N14, so some tags will have two -1 peaks. It gets complicated, and that is why I do not try to correct for this. I just assume that I have about one order of magnitude effective dynamic range for expression changes. I think that this is the lesser of two evils.

## Conclusions

The evidence over the years has led to the idea that non-specific background noise isolated with the precursor of interest is what accounts for the ratio distortion in isobaric labeling. That is a dynamic and complex problem. The noise is non-specific carry over from other (labeled) peptides that have very similar m/z to the precursor of interest. Carry over can come from a dirty instrument, overloaded LC components, how m/z values are distributed in your sample (some m/z values may be "busier" than others), or other aspects of particular samples. Trying to generalize or estimate the noise level might be difficult. We also saw from the mocked up tables above that the relative abundance of the reporter ions relative to the noise level is also a very important factor.

My recommendation from looking at quite a few datasets is to use the SPS MS3 method if you have access to Tribrid instruments. If you do not have a Tribrid instrument and the samples are important, it might be better to send samples to someone who has a Tribrid instrument rather than use a Q-Exactive platform. Large scale expression measurements (Next Gen sequencing or proteomics) often produce large numbers of potential candidates and distinguishing between biologically significant and mathematically significant candidates is critical. The wider dynamic range and better accuracy of SPS MS3 TMT data compared to MS2 TMT data would be important to be able to use a fold-change cutoff as a distinguishing method.

The noise background in MS2 data does not just alter the expected ratios. It reduces the difference in the sample means (which negatively affects statistical testing), but it also reduces the variance in the data (which goes in the opposite way for statistical testing). If relative variance decreases faster than the difference in means, it could lead to false positives. This could make validating the results harder and more expensive.   

---

-Phil Wilmarth, OHSU, 20200105    
