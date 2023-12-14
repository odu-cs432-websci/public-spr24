# Homework 2 - Ranking Webpages
**Due:** Sunday, February 11, 2024 by 11:59pm

Read through the entire assignment before starting.  *Do not wait until the last minute to start working on it!* 
 
## Assignment

This assignment will have you investigate different ways to rank webpages based on a query.  

Write a report that answers and *explains how you arrived at the answers* to the following questions.  Be sure to address any questions that are asked (indicated by "*Q: ...?*" in italics). Include any interesting findings that you discover from your analysis.
 
Before starting, review the [HW report guidelines](getting-started/reports.md).  Name your report for this assignment `HW2-report` with the proper file extension. 

**Reminder about Programming Tasks:** For several of the programming tasks this semester, you will be asked to write code to operate on 100s of data elements.  If you have not done this type of development before, I *strongly encourage* you to start small and work your way up.  Especially when you are using new tools, start on a small test dataset to make sure you understand how to use the tool and that your processing scripts are working before ramping up to the full set. *This will save you an enormous amount of time.*

**Important:** This assignment requires the use of URIs collected in HW1. If you did not complete HW1 satisfactorily, contact me for instructions on how to proceed. This **cannot** be done at the last minute.

### Q1. Data Collection

*For the following tasks, consider which items could be scripted, either with a shell script or with Python.  You may even want to create separate scripts for different tasks.  It's up to you to determine the best way to collect the data.*

Download the HTML content of the 500 unique URIs you gathered in HW1 and strip out HTML tags (called "boilerplate") so that you are left with the main text content of each webpage.  ***Plan ahead because this will take time to complete.***

Note: If you plan completing this question in Windows PowerShell (instead of Linux), you will need to be aware of how PowerShell [uses character encoding for string data](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_character_encoding?view=powershell-7.1) (see also [Understanding file encoding in VS Code and PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/dev-cross-plat/vscode/understanding-file-encoding?view=powershell-7.1)).

#### Saving the HTML Files

We just want to save the HTML content.  In Python, we can use the [requests library](https://requests.readthedocs.io/en/latest/) to download the webpage. Once the webpage has been successfully requested, the [HTML response content can be accessed using the `text` property](https://requests.readthedocs.io/en/latest/user/quickstart/#response-content).

You'll need to save the HTML content returned from each URI in a uniquely-named file.  The easiest thing is to use the URI itself as the filename, but your shell will likely not like some of the characters that can occur in URIs (e.g., "?", "&").  My suggestion is to hash the URIs to associate them with their respective filename using a [cryptographic hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function), like MD5.  

For example, https://www.cnn.com/world/live-news/nasa-mars-rover-landing-02-18-21 hashes to `2fc5f9f05c7a69c6d658eb680c7fa6ee`:
```console
$ echo -n "https://www.cnn.com/world/live-news/nasa-mars-rover-landing-02-18-21" | md5sum | awk '{print $1}'
2fc5f9f05c7a69c6d658eb680c7fa6ee
```
Notes:
* `md5sum` might be `md5` on some machines
* note the `-n` option to `echo` -- this removes the trailing newline
* `awk '{print $1}'` at the end prints only the characters before the first space in the output (i.e., the hash) -- *try the command without this to see the difference*

If you want to use Python for this, you can use the [hashlib library](https://docs.python.org/3/library/hashlib.html). Note that you'll want to strip any trailing whitespace or newline characters from the URI (using [`strip()`](https://www.w3schools.com/python/ref_string_strip.asp)) before you compute the MD5 hash.

For later analysis, you will need to map the content back to the original URI, so make sure to save a text file that contains all of the URI to hash mappings.

#### Removing HTML Boilerplate

Now use a tool to remove (most) of the HTML markup from your 500 HTML documents. 

The Python boilerpy3 library will do a fair job at this task.  You can use `pip` to install this Python package in your account on the CS Linux machines.  The [main boilerpy3 webpage](https://pypi.org/project/boilerpy3/) has several examples of its usage.

Keep both files for each URI (i.e., raw HTML and processed), and upload both sets of files to your GitHub repo. Put the raw and processed files in separate folders.  Remember that to upload/commit a large number of files to GitHub, [use the command line](https://docs.github.com/en/github/managing-files-in-a-repository/adding-a-file-to-a-repository-using-the-command-line).

Sometimes boilerpy3 isn't able to extract any useful information from the downloaded HTML (either it's all boilerplate or it's not actually HTML), so it produces no output, resulting in a 0B size file.  You may also run into HTML files that trigger UnicodeDecode exceptions when using boilerpy3.  You can skip files that have  these types of encoding errors, result in 0B output, or contain inappropriate content (whatever you define as such).  The main goal is to have enough processed files so that you can find 10 documents that contain your query term (for Q2 and later).

*Q: How many of your 500 URIs produced useful text?  If that number was less than 500, did that surprise you?* 

### Q2. Rank with TF-IDF

Choose a query term (e.g., "coronavirus") that is not a stop word (e.g, "the"), not super-general (e.g., "web"), and not used in HTML markup (e.g., "http") that is found in at least 10 of your documents.  If the term is present in more than 10 documents, choose any 10 English-language documents from *different domains* from the result set.  (Hint: You may want to use the Unix command `grep -c` on the processed files to help identify a good query -- it indicates the number of lines where the query appears.) 

As per the example in the [Searching the Web slides](https://docs.google.com/presentation/d/1xHWYidHcqPljtvqcGsUXgXU7j6KEFDVXrTftHmkv6OA/edit?usp=sharing), compute TF-IDF values for the query term in each of the 10 documents and create a table with the TF, IDF, and TF-IDF values, as well as the corresponding URIs. (If you are using LaTeX, you should create a [LaTeX table](https://www.overleaf.com/learn/latex/tables).  If you are using Markdown, view the raw version of this file for an example of how to generate a table.) Rank the URIs in decreasing order by TF-IDF values.  For example:

Table 1. 10 Hits for the term "shadow", ranked by TF-IDF.

|TF-IDF	|TF	|IDF	|URI
|------:|--:|---:|---
|0.150	|0.014	|10.680	|http://foo.com/
|0.044	|0.008	|10.680	|http://bar.com/

You can use Google or Bing for the DF estimation:
* Google - use **40 billion** as the total size of the corpus
* Bing - use **4 billion** as the total size of the corpus

*These numbers are based on data from <https://www.worldwidewebsize.com>.*

To count the number of words in the processed document (i.e., the denominator for TF), you can use the Unix command `wc`:

```console
% wc -w 2fc5f9f05c7a69c6d658eb680c7fa6ee.processed
    19261 2fc5f9f05c7a69c6d658eb680c7fa6ee.processed
```
It won't be completely accurate, but it will be probably be consistently inaccurate across all files.  You can use more 
accurate methods if you'd like, just explain how you did it.  

Don't forget the log base 2 for IDF, and mind your [significant digits](https://en.wikipedia.org/wiki/Significant_figures#Rounding_and_decimal_places).

*You must discuss in your report how you computed the values (especially IDF) and provide the formulas you used for TF, IDF, and TF-IDF.*  

### Q3. Rank with PageRank

Now rank the *domains* of those 10 URIs from Q2 by their PageRank.  Use any of the free PR estimators on the web,
such as:
* https://searchenginereports.net/google-pagerank-checker
* https://dnschecker.org/pagerank.php
* https://smallseotools.com/google-pagerank-checker/
* https://www.duplichecker.com/page-rank-checker.php

Note that these work best on domains, not full URIs, so, for example, submit things `https://www.cnn.com/` rather than `https://www.cnn.com/world/live-news/nasa-mars-rover-landing-02-18-21`.

If you use these tools, you'll have to do so by hand (most have anti-bot captchas), but there are only 10 to do.  

Normalize the values they give you to be from 0 to 1.0.  Use the same tool on all 10 (again, consistency is more important than accuracy). 

Create a table similar to Table 1:

Table 2.  10 hits for the term "shadow", ranked by PageRank of domain.

|PageRank	|URI
|-----:|---
|0.9|		http://bar.com/
|0.5	|	http://foo.com/

*Q: Briefly compare and contrast the rankings produced in Q2 and Q3.*

## Extra Credit

### Q4. *(2 points)* 
Compute the [Kendall Tau_b score](https://en.wikipedia.org/wiki/Kendall_rank_correlation_coefficient#Tau-b) for the lists from Q2 and Q3 (use "b" because there will likely be tie values in the rankings). Report both the Tau value and the "p" value.

### Q5. *(3 points)*  
Build a simple (i.e., no positional information) inverted file (in ASCII) for all the words from your 500 URIs.  Upload the entire file your GitHub repo and discuss an interesting portion of the file in your report.

## Submission

You should be working in the private GitHub repo that I created for you in the [odu-cs432-websci organization](https://github.com/odu-cs432-websci/) (your repo URL should look something like https<nolink>://github.com/odu-cs432-websci/spr24-*username*/). 

If you are working locally, make sure that you have committed and pushed your local repo, including `HW2-report.md` or `HW2-report.pdf` (and `HW2-report.tex`) and any images you reference, to GitHub. 

Submit the URL of your report (*not the URL of your repo*) in Canvas under HW2. This should be something like  
https<nolink>://github.com/odu-cs432-websci/spr24-*username*/blob/main/HW2-report.{md,pdf}

*If you make changes to your report after submitting in Canvas, I will use the last commit time in your repo as your assignment submission time.*
