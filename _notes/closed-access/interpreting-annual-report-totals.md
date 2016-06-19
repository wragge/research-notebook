---
date: 2016-06-10
layout: default_note
note_type: Hacking
title: Interpreting annual report totals
category: Closed Access
date: 2016-06-10
---

I've extracted [access examination statistics]({{ site.baseurl }}/notes/access-examination-statistics/) from NAA Annual Reports. The question now is how do these stats compare to the data I've harvested about closed files from RecordSearch.

In theory, the 'Withheld from public access' number in the annual reports should be greater than or equal to the number of records in my database with access decisions made within the financial year. There may be fewer records in my database because (hopefully) some files have been opened since the original decision was made, but there shouldn't be more.

This is complicated by the fact that it's not clear how the totals in the Annual Reports were arrived at. The 2011-2012 Annual Report notes that:

> For 2011–12 all data was sourced from the reference and access module of RecordSearch. In previous years a combination of different electronic databases was used to collate the statistics.

Let's have a look...

<iframe class="img-example" width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~wragge/424.embed"></iframe>

{: .table}
| Financial year | Annual Report | ClosedAccess |
|---|---|---|
|1996-1997|165|[273](https://closedaccess.herokuapp.com/items/?q=&decision_after=1996-07-01&decision_before=1997-06-30&sort=decisions)|
|1997-1998|103|[209](https://closedaccess.herokuapp.com/items/?q=&decision_after=1997-07-01&decision_before=1998-06-30&sort=decisions)|
|1998-1999|227|[302](https://closedaccess.herokuapp.com/items/?q=&decision_after=1998-07-01&decision_before=1999-06-30&sort=decisions)|
|1999-2000|172|[101](https://closedaccess.herokuapp.com/items/?q=&decision_after=1999-07-01&decision_before=2000-06-30&sort=decisions)|
|2000-2001|159|[105](https://closedaccess.herokuapp.com/items/?q=&decision_after=2000-07-01&decision_before=2001-06-30&sort=decisions)|
|2001-2002|123|[114](https://closedaccess.herokuapp.com/items/?q=&decision_after=2001-07-01&decision_before=2002-06-30&sort=decisions)|
|2002-2003|739|[616](https://closedaccess.herokuapp.com/items/?q=&decision_after=2002-07-01&decision_before=2003-06-30&sort=decisions)|
|2003-2004|345|[215](https://closedaccess.herokuapp.com/items/?q=&decision_after=2003-07-01&decision_before=2004-06-30&sort=decisions)|
|2004-2005|171|[112](https://closedaccess.herokuapp.com/items/?q=&decision_after=2004-07-01&decision_before=2005-06-30&sort=decisions)|
|2005-2006|245|[204](https://closedaccess.herokuapp.com/items/?q=&decision_after=2005-07-01&decision_before=2006-06-30&sort=decisions)|
|2006-2007|299|[125](https://closedaccess.herokuapp.com/items/?q=&decision_after=2006-07-01&decision_before=2007-06-30&sort=decisions)|
|2007-2008|448|[290](https://closedaccess.herokuapp.com/items/?q=&decision_after=2007-07-01&decision_before=2008-06-30&sort=decisions)|
|2008-2009|267|[109](https://closedaccess.herokuapp.com/items/?q=&decision_after=2008-07-01&decision_before=2009-06-30&sort=decisions)|
|2009-2010|250|[134](https://closedaccess.herokuapp.com/items/?q=&decision_after=2009-07-01&decision_before=2010-06-30&sort=decisions)|
|2010-2011|289|[381](https://closedaccess.herokuapp.com/items/?q=&decision_after=2010-07-01&decision_before=2011-06-30&sort=decisions)|
|2011-2012|285|[1233](https://closedaccess.herokuapp.com/items/?q=&decision_after=2011-07-01&decision_before=2012-06-30&sort=decisions)|
|2012-2013|239|[384](https://closedaccess.herokuapp.com/items/?q=&decision_after=2012-07-01&decision_before=2013-06-30&sort=decisions)|
|2013-2014|659|[1742](https://closedaccess.herokuapp.com/items/?q=&decision_after=2013-07-01&decision_before=2014-06-30&sort=decisions)|
|2014-2015|1,026|[1572](https://closedaccess.herokuapp.com/items/?q=&decision_after=2014-07-01&decision_before=2015-06-30&sort=decisions)|

Ok, so there's odd things going on at either end of this time period -- the number of files 'withheld from public access' according to the annual reports is less than the number of closed files (on 1 January 2016) that have an access decision within the financial year. I suspect that there's two separate causes for this discrepancy.

If you look at the access examination statistics for the years 1996-7 to 2003-4 you'll notice they had an extra category called either 'items containing no open period material' or 'other (eg closed period)'. If you look at my harvested data you'll see that it includes 'Closed Period' files for these years. So it seems reasonable to add the 'withheld from public access' figures to the 'other' category. If we do that we see:

<iframe class="img-example" width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~wragge/425.embed"></iframe>

Yikes! The early years are now totally out of kilter with everything else. This seems to indicate that the 'other' category doesn't just include closed period files.

At the other end of the period, I suspect that the big differences between what's reported and what I've harvested are due to the non-reporting of records in the 'withheld pending advice' category. If I exclude these from my data we see:

{: .img-example}
<iframe width="100%" height="400" frameborder="0" scrolling="no" src="https://plot.ly/~wragge/427.embed"></iframe>

With the exception of 2013-2014, this seems to produce a reasonable-looking comparison. I think this is good evidence that the statistics in annual reports don't include records that are 'withheld pending advice'. Is this reasonable? What does this mean for the transparency of the access examination process? I think I need to explore this further elsewhere.

What was going on in 2013-2014? If you look at the other reasons records were closed in that period you'll see there were lots of 'Parliament Class A' records (1034 of them!). Are these included in the annual report stats? I think some more poking around in 2002-2003 and 2013-2014 is required.

What have I learnt? -- that there's no straightforward way of reading the annual report statistics.