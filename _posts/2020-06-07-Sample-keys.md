---
layout: post
title: "Sample Keys"
date: 2020-06-07
---

# Sample Keys for Data Repositories

Have you fetched some cool data from one of the data repositories and stared to re-analyze it? What was the first snag you ran into? Could you relate the instrument file names to the study described in the paper? Could you tell which fractions went together? Could you tell which mass spec samples were which biological samples? If there were **hidden** quantitation channels (like TMT), did you know what samples were in which channels? Was the necessary information in the paper, in the repository description, as a file in the actual repository, or missing?

What you really wanted was **the sample key**. That sounds really simple, right? Well, buckle up folks. Let's talk about how simple it is to make a sample key for a proteomics experiment.

## Experimental designs:

This is a higher-level conceptual description of the problem that the [SDRF proposal](https://github.com/bigbio/proteomics-metadata-standard) addresses. The goal is to have a way to make sure some additional information about study designs is captured so that re-analysis of project can actually happen.

## Overview:

What we want at the highest conceptual level is to know the relationship between an individual instrument file and the biological experiment. I think there is some consensus that describing biological experiments are hard because they take many forms. There is an incorrect perception that there is little ambiguity about what an individual instrument file represents. An individual biological sample is an entity in a multi-dimensional space. A single instrument file is also an entity in a multi-dimensional space. We need to define those spaces before we can think about how to map them to each other.

A biological experiment can take many forms. It is seldom just a simple collection of biological samples. There are comparative experimental designs that take many forms. There can be one or more disease states and controls (two sample tests, single factor ANOVA, etc.). There can be repeated measure studies (before and after treatments, before and after illness, etc.). There can be time courses or temperature course (samples can be repeated measures or independent).

How to test the data and interpret the results requires knowledge of the experimental design. Sample labels are also affected by design. A basic requirement, since we use computers for analysis, is a unique designation for each sample.

## How to represent sample names:

Ways to represent samples can take many different forms. Samples can be assigned a unique key (like SQLite3 databases). Samples could have a joint or compound key (again using relational database language). For repeated measure designs, the samples are the same for the repeated measures and the sample designation might be the same at each repeated measure. How do we take one or more identifiers associated with a biological sample and make a unique identifier? There are many established options here.

You can think about relational data tables and define a `sample table`. Each record (row) in a sample table would have a set of fields (cells) to describe the sample. Commonly, each field would have a defined type and each record would be unique. Since we would need other tables to describe the experimental design and the data structure for the experiment, we would also need a key for each record to link tables to each other. The key could be an explicit field, such as a unique numerical value (SQLite3 uses an automatically assigned integer key in an extra table field). There could be a field that is already unique and could serve as a key. A combination of fields might function as a unique key. An explicit unique string could also be constructed to serve as a key. Methods where the computer does the bookkeeping can be more reliable at the expense of human readability.

There are serialization techniques for encapsulating information that has attributes so that a data item can be transferred from computer to computer reliably in a more stand-alone way. Markup languages like HTML are designed to let browsers (clients) communicate with servers. This allows simultaneous transmission of a variety of data types in an asynchronous fashion. There is overhead to package the items with their attributes and to decode the items after transmission. XML is an extension of HTML and has been widely used (maybe misused is more accurate) in proteomics file standards. XML is a poor way to encapsulate tables of numerical data, for example.

JSON is a more concise way to store (or send) labeled information than XML. JSON and Python dictionaries are conceptually very similar and very easy to interconvert. If you wanted to think of how JSON and a relational database table compare, you could think of JSON kind of like saving (or sending) each row of a table together with its column headers. JSON is flexible and empty fields (cells) can just be omitted. If you like to think about computer efficiency, a large table with few missing cells would take much more space (or time) to have each row with its own column headers. If the table is sufficiently sparse, then dictionaries or JSON can be more efficient.

A common requirement for any labeled data is that the labels are fully defined. This is true for column headers in tables, field names in database records, tags in markup languages, and keys in dictionaries. Relational databases and markup files should have schemas. Data tables should have associated column key tables. This is something that can be easily overlooked.

## Biological samples:

A possible way to generalize sample designations might be using a defined [tuple](https://en.wikipedia.org/wiki/Tuple) nomenclature.  That requires definition of information that depends on the experimental design. A simpler design would need less information for each sample. A collection of tuples needs each tuple to be the same length and have the tuple items in the same order. Tuples can have mixed data types (strings, integers, floating point, Boolean, etc.). You could think of sample tuples as coordinates in some multi-dimensional biological study design space.

A somewhat general format could be: (biological subject designation, experimental condition designation, [optional repeated measure designations]). The overall tuple would need to be unique experiment-wide; however, individual elements of the tuple do not have to be unique. Biological replicate/subject designations could be any set of labels, although unique labels within an experimental condition might be easier to conceptualize. Examples could be: `1, 2, 3`; `A, B, C`; `Control-1, Control-2, Control-3`; etc. There are always arguments for and against redundancy in definitions. Redundancy can lead to errors and maintenance issues. Strict non-redundancy may be harder for human readability.

## Mass spectrometry data:

In much the same way that a tuple can define a sample in a high dimension study design space, a mass spec file is a coordinate in an analytical measurement space. A particular mass spec data file is interesting. It is an individual item of a larger space (a coordinate in some sample and separation space) that also contains items from smaller spaces (a coordinate in additional separation spaces [ion mobility, retention time], m/z space [MS2 scans, MS3 scans], and measurement space [m/z and intensity pairs]). We can ignore the structures within an individual instrument file for this discussion.

We do have to think about the larger separation space that the file is part of. We have some collection of starting “samples”, typically in Eppendorf tubes. Those “samples” may not be individual biological samples because we have several stable isotope labeling methods that involve multiplexing of samples (2 or 3 samples in SILAC, and a variety of larger multiplex options for isobaric labeling). These MS samples can be intact proteins or peptides from digested proteins. Each of those types of samples can be fractionated. The fractionation may be external to the HPLC system and ion source connected to the mass spectrometer, or may be integrated into the analytical platform.

Protein separations are becoming less common. These separation can use liquid chromatography (size exclusion, strong anion, C4 reverse phase, etc.) or gel electrophoresis (2D gels, 1D gel lanes, 1D gel run-ins, etc.). Protein separations are followed by enzymatic digestion for shotgun proteomics.

Peptide digests can be separated into one or more fractions to spread out the complex peptide mixtures to reduce dynamic range issues from abundant signals and to allow more instrument time to characterize the sample. Multi-dimensional chromatography separations can be added upstream of the (typical) final low pH reverse phase C18 column separation (there are other types of columns, too). The upstream separations (commonly referred to as the first dimension) can be off-line or on-line with the final separation (the second dimension). Off-line usually denotes a separate chromatography separation with fraction collection. The chromatography can be strong cation exchange (SCX), isoelectric focusing, or high pH reverse phase. Collected fractions may be combined so that the final number of first dimension “fractions” is reduced. The number of first dimension fractions commonly range from 8 to 24 with current methods.

Some types of first dimension separations can be done in-line with the final C18 separation. SCX separations in bi-phasic columns are used in the classic MudPIT method. Many HPLC systems support multiple buffers and pumps so that larger peptide samples can be “bumped” off appropriate substrates in smaller batches, captured by the C18 column, and eluted into the mass spectrometer.

The coordinates of the separation of the mass spectrometry sample are often encoded in the instrument file name, but a separate record of the mapping can also be used. Downstream data analysis needs to know which multi-dimensional separation fractions constitute each mass spectrometer “sample”. The serial order of the fractions may also be important for analysis interpretation (carryover of contaminants or abundant analytes) or other analysis steps (match between runs, chromatography alignments) and may need to be logged. Ultimately, data from fractions will frequently be aggregated back to the mass spec “sample”.

## Relationship between mass spec runs and biological samples:

We have a complicated collection of instrument files produced by the biological study. Sets of instrument files are associated with specific mass spectrometry samples (a set of instrument files are the first-dimension fractions from a single mass spec sample). Each mass spec sample can contain one or more biological or technical samples. (How are the technical replicates used in the data analysis? How is the technical replicate data aggregated for each biological replicate? These complications are compelling arguments to not do technical replicates.) Specific biological samples will be members of sets of experimental conditions that make up the biological experiment. A biological study may have more than one biological experiment. This is a complicated collection of one-to-many, many-to-one, or many-to-many mappings.

Some sort of flat file representation of these relationships might be difficult to create. It seems more naturally like a relational database scheme. Maybe even two sets of relational tables that can be connected via foreign keys. One set of tables would describe the biological samples and how they are organized into the biological study. The other set of tables would describe the mass spectrometry instrument files and how they are organized into the mass spectrometry “samples”. The foreign key relationship would be to define how the final biological samples relate to the starting mass spectrometry samples.

Chronological ordering might help conceptualize things. We mostly have increasing numbers of things, but it is not monotonically increasing. We might have a study idea. It would compare controls to treatments. We would have some number of replicate samples for each condition. Those samples would be digested and fractionated. Each fraction would be an LC run. Most of these steps increase things. If we increase the number of biological replicates in each condition, we will have more LC runs. If we increase the fractionation, we have more LC runs. Multiplexing methods condense things in limited ways at specific steps.

We also have some hidden and extraneous dimensions to deal with. You can think of sample multiplexing as hiding some of the biological sample dimension. You can think of multi-dimensional fractionation in separations as extraneous dimensions. We don’t really want to add fractions because they cost us time and money. We have to do it to compensate for current instrument limitations.

## Observations:

It is not clear how far upstream to go with study design details. There needs to be enough detail to make sure any statistical analyses are not done improperly. I am not exactly sure what level of detail that implies. It is clear that technical replicates have to be clearly identified so that they are not mis-interpreted as biological replicates. Which samples belong in which experimental conditions is also needed. Details about any repeated measures aspect to a study (paired studies, etc.) is important to prevent loss of statistical power. How the biological conditions relate to each other may be better left to the publication. At some point, you have to bite the bullet and understand why the experiment was done and what questions it was trying to answer.

It is not clear if any specific details about the mass spectrometer are needed. That information is already captured in data repository submissions. For example, the mapping of biological samples to isobaric tag reporter ion masses is needed, but how the instrument produces reporter ions (MS2 scans or SPS MS3 scans) is not needed (at least not here). I don't ever work with DIA data, so I have not thought about if all of this only applies to DDA data. I suspect most of this applies to discovery, targeted, DDA, and DIA experiments.

---

Phil Wilmarth
June 7, 2020
