---
layout: post
title: "Is proteomics like house building?"
date: 2020-01-06
---

## How do protein databases and protein inference frame proteomics results?

We have all been there. The holiday dinners with family and friends. The inevitable question, "so what do you do?" We all struggle for some way to explain research to non-researchers. Maybe this house building analogy for proteomics can help.

The peptide digest and mass spec data before any analysis is kind of like having all of the possible materials you could use to build a house. We do not really have all possible outcomes (it is not like Schrodinger’s cat), but our imagination is pretty unconstrained at the start. When we have sample knowledge like the species, then limiting results to the experimental constraints is like setting the budget and picking the Home Depot instead of Lowes (two building supply store chains). You are now limited to only what that store can offer; however, it is still a big store.

We do not buy everything at the store, though. We have a budget and we need a shopping list. Defining the general categories (some lumber, some roofing, some electrical, some plumbing, etc.) is like configuring the database search. The database search results are like making the detailed shopping list. These are all of the parts we need to buy to make our house. Imagine some big trucks come to the house site and drop off all of the parts we bought.

What is different with proteomics (compared to house building) is that we get all of the parts without having a house plan or blueprints. We have to unpack the parts, sort through them, and group things. We have to start imagining what kind of house we can make out of the parts. If we have two toilets, we might think two bathrooms. We might have a lot of possibilities. We might have some building codes to reduce the possibilities. We might have some ranges on room sizes, or rules about wiring and plumbing. Having some experience assembling houses out of pile of parts might be helpful, too.

Different FASTA protein files (there are almost always multiple choices for a given species) results in slightly different parts lists, so we might have some slightly different resulting house plans that can accommodate the parts. (To be more accurate, the protein FASTA file affects the parts list AND imposes some constraints on the house designs – it is more circular than our house building analogy). Our house (the inferred proteins) will have some dependence on the parts list choices (the FASTA files).

We have examined our parts list and figured out a house plan to construct. We start building. We end up finding out that we forgot some things. We have the frame, some walls, a roof, windows, doors, etc. However, we realize that we overlooked a lot of finish details. We forgot the paint and some of the flooring. We did not get any towel bars for the bathrooms. The kitchen does not have all of the appliances. All of these other “little things” are the actual biological information we are trying to use proteomics methods to figure out. You know, the things like quantification, protein interactions, PTMs, etc. These are things that we decorate the house with. We always have to create the house before we can decorate it.

The protein FASTA file and the protein inference rules determine the layout, the foundation, and the frame for the house. Everything else has to hang from that framework. A stronger foundation and solid framing will result in a better final house.

I think too many times the upstream analysis steps are not explored well enough in proteomics. Different FASTA files can be tried. Is the FASTA file complete enough for the questions being asked? Is the FASTA file too complicated for the experiment? Different search programs or setting can be tried. Are there semi-tryptic peptides? Are there abundant PTMs that are biologically significant? These questions affect the downstream steps and should be resolved before interpretation and validation of results are done. It is like looking at some house plans and picking a good one before building the house.    

---

-Phil Wilmarth, OHSU, 20200106    
