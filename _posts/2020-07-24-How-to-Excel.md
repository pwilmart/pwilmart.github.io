---
layout: post
title: "How to Excel"
date: 2020-07-24
---

## Addressing a couple of questions:

1. Why do I use an Excel spreadsheet to summarize proteomics results?

1. Why do I horizontally stack the statistical test results from different comparisons into one sheet?

---

### Why do I use Excel spreadsheets?

My pipeline creates tab-delimited text files like many other pipelines. We start with those files and polish them up a bit with Excel to send to clients. We add statistical testing results to the proteomics results tables so that everything results-wise is in one place. We do some layout and formatting things to make the sheets a bit more inviting. These files are appropriate for Supplemental files. I think Excel files make excellent deliverables for a core facility and here are my reasons.

I think orders of magnitude more scientists have looked at data tables with Excel (or an open office equivalent) that have loaded data into a data frame in R. I will not try and shame scientists into becoming computer programmers or statisticians. We need scientists to be scientists. I am not a great swimmer. I need to swim downstream here.

Excel can be a double-edged sword. It has no rivals as an interactive table viewer. Its feature set is extensive (too extensive?) and its performance is truly stunning. Memory in computer is one dimensional. Tables are two dimensional. Under the hood, some table operations (let’s say on columns) are fast because the data are together in memory. Operations in the other direction (rows) are harder because the cells are scattered all over memory. Excel is always fast. If you hate Excel (probably because you spent way more time learning how to do something in R) and cannot acknowledge what an outstanding piece of software Excel is, then I have no words.

You can color cells, change cell outlines, change fonts, sizes, formatting, etc. with almost no limitations. There are many ways to help visually organize tables and help make tables more self-documenting. That is nice for scientific use. There are actual reasons that interactive graphical user interfaces took over the world instead of command line interfaces. They are more intuitive and have lower barriers to entry.

The main downside to Excel is that what-you-see-is-what-you-get interfaces are very dynamic. This makes logging a full history of all table operations a challenge. It can be hard to figure out exactly how a sheet was created and even harder to recreate it a second time. This can hinder attempts at scientific reproducibility.

Excel, like Clippy, tries a little too hard to help you out. Most data in cells is parsed to guess what type of data it might be and then auto converted into new formats. If you have a gene name like MAR9, Excel will see your “mistake” and make it March 9th. But you did not supply a year, so Excel guesses. These date conversions are problems in Excel because it does not remember the “MAR9” string. Dates are stored as floating-point numbers. You cannot get “MAR9” back from the floating-point number. You can prevent Excel from converting text strings to other data types, but you have to prevent Excel from doing that before it does that. It is a catch 22.

Excel will not save you from yourself. To do that, you have to adopt some good work habits. Save the sheets frequently when working on them. Make frequent copies of the sheets as you work on them. I might make a “backup” folder, save sheets at key points in time (after any significant changes), add a date stamp to the filename (add a “20200724_” prefix, for example), and move the backup sheets to the folder. You can also make a “log” tab to function like a lab notebook where you can keep notes about what you did to create the sheet. I try to add new columns to sheets at the right of existing columns (sort of a build the sheet from left to right). This can make it easier to recognize the starting data for an analysis (it will be the left side of the sheet).

How to work with large sheets to add additional content takes learning and practice. That is why I recommend making backup copies of sheets. Your first attempts at exploring data and adding content will likely be less than you hope for. You will screw up the sheets in ways that you cannot recover from. You will have to go back to an earlier version and start over. You will learn how to do things in ways that do not break your sheets, but that takes time to learn. I will have some tips and techniques at the end of this document.

### Why do I horizontally stack different comparisons into one sheet?

I think it is good science to have results from statistical testing for every protein that was quantifiable. That means the statistically significant candidates **and** all of the non-significant candidates. What changed in expression is important, but so are all of the things that did not change.

When multiple groups are in an experiment, there should be some “rhyme to the reason”. Multiple groups were part of the experimental design because each tells you something different (but related) to the overarching scientific question. Maybe a process should be more active in one comparison but not changed in another. Proteins that were differential in one comparison may need to be **not differential** in other comparisons. All of the test results horizontally aligned makes such criteria easier to explore.

Conditions may be progressive and seeing progressive changes in protein expression may be key to understanding the molecular mechanisms at play. Having consistently normalized data from multiple conditions in the same table row facilitates finding such proteins.

We know that biological processes involve multiple proteins and that there can be a wide variety of ways that proteins can be involved in processes (increasing an inhibitor can decrease a pathway, for example). We need the results of quantitative proteomics experiments to be interactive and explorable by the biologist to try and find the patterns in the data. Slicing and dicing a data table is a poor way to find data patterns. It is, however, about all we have at this point in time. Because spreadsheets are interactive, visual, and can be added to, they can function as pattern finders, albeit not as efficiently as we would hope. The domain expertise of the experienced scientist is critical in this pattern finding exercise and an ability to manipulate the data by the scientist is essential.

### Tips and tricks

Use a computer with one or more big monitors. You want a lot of screen real estate for this work. I find a mouse is better than a trackpad (your mileage may vary). There are a few sheet viewing options that you should be very comfortable with. Split views are essential for sheet with thousands of rows and dozens (hundreds?) of columns. You need to see the top of the sheet and the bottom of the sheet at the same time if you want to add new columns that are filled with text or are computing something. You may want to view two sets of columns (two different left/right regions), such as results from two comparisons at the same time. Hiding and unhiding columns can also help for some steps. An alternative way to get two sets of columns together (instead of split view) is to hide all of the intermediate columns. From a scientific integrity standpoint, it is better to temporarily hide data than to delete data.

Arrow keys move to the next cells in the directions of the arrows. CONTROL+ARRROW (Command-Arrow on Macs) moves to the ends of cell ranges. That gets you around the sheet more quickly. You can select ranges of cells by clicking in the first cell and then SHIFT+CLICK on the last cell. This is handy for top to bottom column selections when using split views. CONTROL+R (fill right) and CONTROL+D (fill down) are ways to populate ranges of cells with the same text values or the same formula.

Column filters are very useful. You can find tutorial on the web if you have not used column filters in the past. You may want to play with data in a scratch sheet to learn about filters. Filters allow you to show only the table rows that have cells that satisfy some criteria for that specific column. Applying multiple column filters is like a logical AND of the criteria. Column filters can also be used to sort table rows.

Column filters have scope. They apply to rectangular regions of the table that have contiguous data. Blank rows or columns inside your data table region will limit the scope of column filters and may not give the results you expect. If you want to be adding new columns to the right of a table, it is wise to add a dummy character (I use X) as a column header in the row that will be selected when filters are activated. I put an X in a column several columns to the right of where the table ends and then turn on filters for that row.

Data exploration is a time-consuming manual exercise. You want to be able save your efforts. I insert columns to hold manually (or computationally) generated text labels. I already have label columns for expression changed direction and DE candidate status when adding statistical testing results. The labels, in conjunction with column filters, let you hone in on the proteins of interest.

Adding your own text columns are particularly useful for function/pathway annotations. Annotations are messy and redundant. Many proteins have multiple functions or locations in cells and the annotations can be a mash up of all of that (sometimes contradictory) information. Your domain expertise can be leveraged to define a smaller set of relevant terms to cover the biological question at hand. For proteins meeting some criteria (maybe statistically significant and more than 2-fold up regulated, for example), the complicated annotation column information can be perused and a simpler, summary term manually added to the new text column. Then that new column can be used to filter proteins by the more relevant terms. These methods may be helpful to add knowledge to the results and formulate biological hypotheses.

---

Phil Wilmarth
July 24, 2020
