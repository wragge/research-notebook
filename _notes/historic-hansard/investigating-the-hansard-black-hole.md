---
date: 2016-05-29
layout: default_note
note_type: Hacking
title: Investigating the Hansard black hole
category: Historic Hansard
---

## Background

While harvesting XML files from Senate Hansard I noticed that some of the downloaded files had differently formatted filenames and were very small (around 300 bytes). Sure enough when I opened them I found they were empty:

``` xml
<?xml version="1.0" ?>
<hansard xsi:noNamespaceSchemaLocation="../../hansard.xsd" version="2.1" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"><session.header>
<date>1907-09-20</date>
<parliament.no>3</parliament.no>
<session.no>2</session.no>
<period.no></period.no>
<chamber>Senate</chamber>
<page.no>3556</page.no>
<proof>0</proof>
</session.header>
</hansard>
```

At first I thought they were just some artefact of the system and didn't really worry about them. But as they started piling up, I thought I'd better investigate.

## What's on ParlInfo

If you search ParlInfo for the date in the XML file above, [20 September 1907](http://parlinfo.aph.gov.au/parlInfo/search/summary/summary.w3p;adv=yes;orderBy=date-eLast;page=0;query=Date%3A20%2F09%2F1907%20%3E%3E%2020%2F09%2F1907%20Dataset%3Ahansards,hansards80;resCount=Default), just one result is returned. This is odd -- normally you'd expect to see a list of the debates and questions that made up the day's proceedings. For example the previous day, [19 September 1907](http://parlinfo.aph.gov.au/parlInfo/search/summary/summary.w3p;adv=yes;orderBy=date-eLast;page=0;query=Date%3A20%2F09%2F1907%20%3E%3E%2020%2F09%2F1907%20Dataset%3Ahansards,hansards80;resCount=Default), returns 75 results.

If you click on the single result for 20 September 1907, you'll notice again that there isn't the usual nested list of the day's proceedings in the left hand column. It's just empty. If you click on the 'View/Save XML' link you'll see the same empty XML file I harvested. The 'Download fragment' link provides an empty PDF template. Interestingly, if you click on the 'Download full day's Hansard' you get a PDF file that says it has 30 pages, but only the first page is visible.

Ok, so something clearly has gone wrong in the scanning/OCR/markup process. These things happen. The question is, what is the impact on users who have assumed that ParlInfo provides a full record of Hansard.

## Is it searchable?

After poking around some more I noticed that some days did seem to have complete PDFs. I'm not sure what the ParlInfo search is actually searching, so I thought I'd better try a few tests to see whether the content of these empty or malformed files were searchable.

As noted above, you can find them by searching for a date range. But what happens when you add a keyword to that search?

The first page of the malformed PDF included a question on the price of starch. So let's add the keyword 'starch' to our query for 20 September 1907 -- no results. Let's check that our query's ok by [broadening the date range](http://parlinfo.aph.gov.au/parlInfo/search/summary/summary.w3p;adv=yes;orderBy=date-eLast;page=0;query=starch%20Date%3A20%2F09%2F1907%20%3E%3E%2031%2F09%2F1907%20Dataset%3Ahansards,hansards80;resCount=Default) to the end of September. This time we get four results -- it seems that the question on the [price of starch](http://parlinfo.aph.gov.au/parlInfo/search/display/display.w3p;adv=yes;db=HANSARD80;id=hansard80%2Fhansards80%2F1907-09-25%2F0009;orderBy=date-eLast;page=0;query=starch%20Date%3A20%2F09%2F1907%20%3E%3E%2031%2F09%2F1907%20Dataset%3Ahansards,hansards80;rec=1;resCount=Default) was asked again on 25 September 1907.

So it seems that content from the 'empty' days is not searchable. **They are effectively missing from ParlInfo searches.**

## The scale of the problem

How many days are missing? I modified my harvesting script to look for the strangely formatted filenames and write the dates and xml file urls to a new file. From 1901 to 1965, there seem to be 94 days where the proceedings are unsearchable. You'll find the full list at the bottom of this page.

On the APH website, I found a page that listed the [number of sitting days per year](http://www.aph.gov.au/Parliamentary_Business/Statistics/Senate_StatsNet/General/sittingdaysyear). I used this to compare the number of sitting days with the number of properly-formatted XML files I'd harvested (there should be one per day) and the 'empty' or unsearchable days. Here are the results:

<iframe width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~wragge/419.embed"></iframe>

You'll notice that in some years the sum of the harvested and empty files doesn't equal the number of sitting days. I've labelled the difference as 'Missing?', but I don't know whether there are additional files missing from ParlInfo or not. It seems more likely that the number of sitting days is wrong, or that there was no Hansard recorded on some sitting days. I won't know for sure until I (or someone else!) has sat down with a hardcopy of Hansard and my list of dates.

But even ignoring the possibly 'missing' files you can see that substantial blocks of Senate Hansard are not being searched by ParlInfo. The impact seems greatest around the WWI period, from 1910 to 1919. In 1917, for example, 21 of 47 sitting days are not being searched.

<iframe width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~wragge/415.embed"></iframe>

Ninety-four days over 65 years might not seem so much. But when you consider that the average number of sitting days per year is 51, you can see that nearly 2 years worth of Hansard is effectively invisible.

So if you've been using ParlInfo to search through Hansard in the period 1910 to 1919 you might want to have a think about what you could have missed!

## What about the House of Reps?

As I noted, I originally wasn't paying much attention to the empty files, and I think there were a couple that popped up when I was harvesting the House of Reps. But I'm pretty sure the scale of the problem is nothing like what I've observed in the Senate. To be sure, I'm running my harvesting script again across the Reps, gathering information about any empty files. So we'll know for sure in a day or two.

## The empty files

Here's the list of the empty files I've identified in the Senate Hansard. I've included links to the XML and PDF versions so you can check for yourselves.

* 20 September 1907 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1907-09-20/toc_unixml/039332190709206_13212.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1907-09-20/toc_pdf/039332190709206.pdf)
* 6 July 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-06/toc_unixml/055332191007063_10787.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-06/toc_pdf/055332191007063.pdf)
* 7 July 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-07/toc_unixml/055332191007074_10789.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-07/toc_pdf/055332191007074.pdf)
* 13 July 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-13/toc_unixml/055332191007133_10793.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-07-13/toc_pdf/055332191007133.pdf)
* 3 August 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-03/toc_unixml/055332191008032_10807.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-03/toc_pdf/055332191008032.pdf)
* 11 August 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-11/toc_unixml/056332191008111_10753.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-11/toc_pdf/056332191008111.pdf)
* 12 August 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-12/toc_unixml/056332191008122_10755.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-12/toc_pdf/056332191008122.pdf)
* 19 August 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-19/toc_unixml/056332191008192_10762.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-19/toc_pdf/056332191008192.pdf)
* 25 August 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-25/toc_unixml/056332191008251_10767.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-08-25/toc_pdf/056332191008251.pdf)
* 1 September 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-01/toc_unixml/056332191009010_10774.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-01/toc_pdf/056332191009010.pdf)
* 7 September 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-07/toc_unixml/056332191009076_10780.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-07/toc_pdf/056332191009076.pdf)
* 13 September 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-13/toc_unixml/057332191009133_10725.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-13/toc_pdf/057332191009133.pdf)
* 29 September 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-29/toc_unixml/057332191009295_10744.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-29/toc_pdf/057332191009295.pdf)
* 30 September 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-30/toc_unixml/057332191009306_10746.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-09-30/toc_pdf/057332191009306.pdf)
* 5 October 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-05/toc_unixml/057332191010054_10749.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-05/toc_pdf/057332191010054.pdf)
* 13 October 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-13/toc_unixml/058332191010133_10697.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-13/toc_pdf/058332191010133.pdf)
* 18 October 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-18/toc_unixml/058332191010181_10700.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-10-18/toc_pdf/058332191010181.pdf)
* 9 November 1910 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-11-09/toc_unixml/059332191011096_10669.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1910-11-09/toc_pdf/059332191011096.pdf)
* 18 October 1911 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-10-18/toc_unixml/061332191110186_10606.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-10-18/toc_pdf/061332191110186.pdf)
* 15 November 1911 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-11-15/toc_unixml/062332191111153_10577.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-11-15/toc_pdf/062332191111153.pdf)
* 22 November 1911 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-11-22/toc_unixml/062332191111223_10584.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-11-22/toc_pdf/062332191111223.pdf)
* 5 December 1911 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-12-05/toc_unixml/063332191112050_10555.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1911-12-05/toc_pdf/063332191112050.pdf)
* 4 July 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-04/toc_unixml/064332191207045_10530.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-04/toc_pdf/064332191207045.pdf)
* 18 July 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-18/toc_unixml/064332191207185_10539.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-18/toc_pdf/064332191207185.pdf)
* 24 July 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-24/toc_unixml/064332191207244_10544.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-07-24/toc_pdf/064332191207244.pdf)
* 2 August 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-02/toc_unixml/065332191208023_13214.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-02/toc_pdf/065332191208023.pdf)
* 7 August 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-07/toc_unixml/065332191208071_10491.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-07/toc_pdf/065332191208071.pdf)
* 14 August 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-14/toc_unixml/065332191208141_10498.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-08-14/toc_pdf/065332191208141.pdf)
* 9 October 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-10-09/toc_unixml/066332191210095_10485.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-10-09/toc_pdf/066332191210095.pdf)
* 11 October 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-10-11/toc_unixml/067332191210115_13653.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-10-11/toc_pdf/067332191210115.pdf)
* 6 November 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-06/toc_unixml/067332191211062_13656.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-06/toc_pdf/067332191211062.pdf)
* 7 November 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-07/toc_unixml/067332191211073_10457.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-07/toc_pdf/067332191211073.pdf)
* 8 November 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-08/toc_unixml/067332191211084_10459.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-08/toc_pdf/067332191211084.pdf)
* 13 November 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-13/toc_unixml/068332191211130_10413.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-13/toc_pdf/068332191211130.pdf)
* 27 November 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-27/toc_unixml/068332191211270_10423.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-11-27/toc_pdf/068332191211270.pdf)
* 5 December 1912 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-12-05/toc_unixml/068332191212051_10433.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1912-12-05/toc_pdf/068332191212051.pdf)
* 9 July 1913 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-07-09/toc_unixml/070332191307092_10337.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-07-09/toc_pdf/070332191307092.pdf)
* 28 August 1913 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-08-28/toc_unixml/070332191308282_11123.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-08-28/toc_pdf/070332191308282.pdf)
* 24 September 1913 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-09-24/toc_unixml/070332191309240_10367.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-09-24/toc_pdf/070332191309240.pdf)
* 29 October 1913 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-10-29/toc_unixml/071332191310295_10322.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-10-29/toc_pdf/071332191310295.pdf)
* 11 December 1913 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-12-11/toc_unixml/072332191312113_10287.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1913-12-11/toc_pdf/072332191312113.pdf)
* 13 May 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-05-13/toc_unixml/073332191405130_10193.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-05-13/toc_pdf/073332191405130.pdf)
* 3 June 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-06-03/toc_unixml/074332191406034_10149.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-06-03/toc_pdf/074332191406034.pdf)
* 18 June 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-06-18/toc_unixml/074332191406185_10163.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-06-18/toc_pdf/074332191406185.pdf)
* 8 October 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-10-08/toc_unixml/075332191410081_10098.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-10-08/toc_pdf/075332191410081.pdf)
* 13 November 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-11-13/toc_unixml/075332191411131_10116.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-11-13/toc_pdf/075332191411131.pdf)
* 27 November 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-11-27/toc_unixml/075332191411271_10128.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-11-27/toc_pdf/075332191411271.pdf)
* 2 December 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-12-02/toc_unixml/075332191412026_10130.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-12-02/toc_pdf/075332191412026.pdf)
* 10 December 1914 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-12-10/toc_unixml/075332191412100_10138.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1914-12-10/toc_pdf/075332191412100.pdf)
* 9 June 1915 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-06-09/toc_unixml/077332191506091_5660.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-06-09/toc_pdf/077332191506091.pdf)
* 19 August 1915 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-08-19/toc_unixml/078332191508196_9994.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-08-19/toc_pdf/078332191508196.pdf)
* 12 November 1915 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-11-12/toc_unixml/079332191511123_9920.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1915-11-12/toc_pdf/079332191511123.pdf)
* 9 May 1916 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1916-05-09/toc_unixml/079332191605096_9922.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1916-05-09/toc_pdf/079332191605096.pdf)
* 22 May 1916 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1916-05-22/toc_unixml/079332191605225_9937.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1916-05-22/toc_pdf/079332191605225.pdf)
* 2 March 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-03-02/toc_unixml/081332191703022_13163.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-03-02/toc_pdf/081332191703022.pdf)
* 14 June 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-06-14/toc_unixml/082332191706144_9790.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-06-14/toc_pdf/082332191706144.pdf)
* 11 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-11/toc_unixml/082332191707113_9792.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-11/toc_pdf/082332191707113.pdf)
* 12 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-12/toc_unixml/082332191707124_13200.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-12/toc_pdf/082332191707124.pdf)
* 13 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-13/toc_unixml/082332191707135_9796.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-13/toc_pdf/082332191707135.pdf)
* 18 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-18/toc_unixml/082332191707183_9797.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-18/toc_pdf/082332191707183.pdf)
* 19 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-19/toc_unixml/082332191707194_9799.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-19/toc_pdf/082332191707194.pdf)
* 25 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-25/toc_unixml/082332191707253_9802.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-25/toc_pdf/082332191707253.pdf)
* 26 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-26/toc_unixml/082332191707264_9804.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-26/toc_pdf/082332191707264.pdf)
* 27 July 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-27/toc_unixml/082332191707275_9806.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-07-27/toc_pdf/082332191707275.pdf)
* 1 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-01/toc_unixml/082332191708012_9808.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-01/toc_pdf/082332191708012.pdf)
* 8 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-08/toc_unixml/082332191708082_9812.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-08/toc_pdf/082332191708082.pdf)
* 9 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-09/toc_unixml/082332191708093_9814.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-09/toc_pdf/082332191708093.pdf)
* 10 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-10/toc_unixml/082332191708104_9816.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-10/toc_pdf/082332191708104.pdf)
* 15 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-15/toc_unixml/082332191708152_9818.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-15/toc_pdf/082332191708152.pdf)
* 16 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-16/toc_unixml/082332191708163_13201.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-16/toc_pdf/082332191708163.pdf)
* 17 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-17/toc_unixml/082332191708174_9820.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-17/toc_pdf/082332191708174.pdf)
* 22 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-22/toc_unixml/082332191708222_9822.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-22/toc_pdf/082332191708222.pdf)
* 23 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-23/toc_unixml/082332191708233_9824.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-23/toc_pdf/082332191708233.pdf)
* 24 August 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-24/toc_unixml/082332191708244_9826.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-08-24/toc_pdf/082332191708244.pdf)
* 25 September 1917 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-09-25/toc_unixml/083332191709255_9782.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1917-09-25/toc_pdf/083332191709255.pdf)
* 22 January 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-01-22/toc_unixml/084332191801222_9729.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-01-22/toc_pdf/084332191801222.pdf)
* 24 January 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-01-24/toc_unixml/084332191801244_9732.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-01-24/toc_pdf/084332191801244.pdf)
* 1 May 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-01/toc_unixml/084332191805013_9750.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-01/toc_pdf/084332191805013.pdf)
* 8 May 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-08/toc_unixml/084332191805083_9756.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-08/toc_pdf/084332191805083.pdf)
* 15 May 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-15/toc_unixml/084332191805153_9761.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-15/toc_pdf/084332191805153.pdf)
* 30 May 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-30/toc_unixml/085332191805302_9704.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-05-30/toc_pdf/085332191805302.pdf)
* 26 June 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-06-26/toc_unixml/085332191806260_9726.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-06-26/toc_pdf/085332191806260.pdf)
* 10 October 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-10-10/toc_unixml/086332191810104_9664.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-10-10/toc_pdf/086332191810104.pdf)
* 19 November 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-11-19/toc_unixml/086332191811191_9692.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-11-19/toc_pdf/086332191811191.pdf)
* 11 December 1918 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-12-11/toc_unixml/087332191812110_9645.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1918-12-11/toc_pdf/087332191812110.pdf)
* 30 July 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-07-30/toc_unixml/088332191907304_9591.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-07-30/toc_pdf/088332191907304.pdf)
* 29 August 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-08-29/toc_unixml/089332191908293_9547.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-08-29/toc_pdf/089332191908293.pdf)
* 17 September 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-09-17/toc_unixml/089332191909170_9555.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-09-17/toc_pdf/089332191909170.pdf)
* 16 October 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-16/toc_unixml/090332191910166_9513.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-16/toc_pdf/090332191910166.pdf)
* 17 October 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-17/toc_unixml/090332191910170_9515.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-17/toc_pdf/090332191910170.pdf)
* 22 October 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-22/toc_unixml/090332191910225_9520.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-22/toc_pdf/090332191910225.pdf)
* 23 October 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-23/toc_unixml/090332191910236_9522.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-23/toc_pdf/090332191910236.pdf)
* 24 October 1919 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-24/toc_unixml/090332191910240_9524.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1919-10-24/toc_pdf/090332191910240.pdf)
* 2 August 1934 -- [XML](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1934-08-02/toc_unixml/144332193408023_6572.xml;fileType=text%2Fxml), [PDF](http://parlinfo.aph.gov.au/parlInfo/download/hansard80/hansards80/1934-08-02/toc_pdf/144332193408023.pdf)





