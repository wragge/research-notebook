---
date: 2016-05-20
layout: default_note
note_type: Hacking
title: Analysing file titles I
category: Closed Access
---

### Code

* [closedaccess-text <i class="fa fa-external-link" aria-hidden="true"></i>](https://github.com/wragge/closedaccess-text) code on GitHub

### Plan

I want to do some analysis of the file titles of closed files -- grouping by reason, series, and year.

I'm intending to work in NLTK and Mallet, but for a first pass I'm just going to generate lists of file titles and throw them into [Voyant <i class="fa fa-external-link" aria-hidden="true"></i>](http://voyant-tools.org/). This should be fun as I've just realised I can pass urls into Voyant's embed widget. So I'm going to try creating a quick page for each reason with a list of titles and a Cirrus word cloud via Voyant.

Steps:

* Write out file titles by reason.
* Create a md page in my notebook for each reason.
* Embed Cirrus widget.

### Writing out file titles

I'll mostly just borrow code from the Closed Access web interface and write the results out to a text file, but:

* it might be interesting to have them in chronological order for some of Voyant's tools?
* file titles often include additions by archivists enclosed in square brackets [like this]. I think I'll remove these for now so I only have the original titles, but some files don't have an original title (too senstive to see!), so I'll lose these completely. I can always change this later.

So, the code:

``` python

import datetime
import os
import re
from pymongo import MongoClient
from credentials import MONGOLAB_URL


DEFAULT_HARVEST_DATE = datetime.datetime(2016, 1, 1, 0, 0, 0)


def make_dir(dir):
    try:
        os.makedirs(dir)
    except OSError:
        if not os.path.isdir(dir):
            raise


def make_data_dir():
    data_dir = os.path.join(os.getcwd(), 'data')
    make_dir(data_dir)
    return data_dir


def get_db():
    dbclient = MongoClient(MONGOLAB_URL)
    db = dbclient.get_default_database()
    return db


def convert_harvest_date(harvest=None):
    try:
        dates = harvest.split('-')
        harvest_date = datetime.datetime(int(dates[0]), int(dates[1]), int(dates[2]), 0, 0, 0)
    except:
        harvest_date = DEFAULT_HARVEST_DATE
    return harvest_date


def get_reasons(harvest=None):
    harvest_date = convert_harvest_date(harvest)
    db = get_db()
    reasons = db.aggregates.find_one({'harvest_date': harvest_date, 'agg_type': 'reason_totals'})
    reason_list = [reason['reason'] for reason in reasons['results']]
    return reason_list


def get_titles(reason=None, series=None, year=None, clean=False):
    data_dir = make_data_dir()
    db = get_db()
    query = {}
    file_title = 'titles'
    if reason:
        query['reasons'] = reason
        file_title = '{}-{}'.format(file_title, reason)
    if series:
        query['series'] = series
        file_title = '{}-{}'.format(file_title, series)
    if year:
        query['year'] = year
        file_title = '{}-{}'.format(file_title, year)
    if clean:
        file_title += '-cleaned'
    records = db.items.find(query).sort([['contents_dates.start_date.date', -1]])
    with open(os.path.join(data_dir, '{}.txt'.format(file_title)), 'wb') as titles:
        for record in records:
            if clean:
                title = re.sub(r'\[.+?\]', '', record['title'])
                title = re.sub(r'\s+', ' ', title).strip()
            else:
                title = record['title']
            titles.write('{}\n\n'.format(title))


def get_titles_by_reason(clean=True):
    reasons = get_reasons()
    for reason in reasons:
        get_titles(reason=reason, clean=clean)

```

Then in iPython console just:

``` python

import collect
collect.get_titles_by_reason()

```

And [the results](https://github.com/wragge/closedaccess-text/tree/master/data).

### Making pages

I've created a markdown page for each reason with just the relevant frontmatter, and a new html layout that pulls in the list of file titles and displays it in a code block.

Having generated the lists of titles and saved them to GitHub, I'm feeding Voyant's Cirrus tool the urls to the raw text files on GitHub. Something like this:

``` http
http://voyant-tools.org/tool/Cirrus/?input=https://raw.githubusercontent.com/wragge/closedaccess-text/master/data/titles-pre-access-recorder-cleaned.txt&stopList=stop.en.taporware.txt&visible=100
```

I'm also telling Voyant to use the English stopwords list (`&stopList=stop.en.taporware.txt`) and to present 100 words by default (`&visible=100`)

Here are the pages for each reason:

{% for reason in site.data.closedaccess-reasons.reasons %}
* [{{ reason }}]({{ site.baseurl }}/experiments/{{ reason | slugify | prepend: 'titles-' | append: '-cleaned/' }})
{% endfor %}

### Problems

The results in Cirrus seem to vary somewhat. I noticed that if I reloaded [33(1)(a)](/experiments/titles-33-1-a-cleaned/) sometimes the most frequent words like 'communist' and 'australia' would disappear. I thought it might be the 'auto' stopwords, so specified the 'Taporware' stopwords, but that didn't seem to help. Changing the default number of words up to 100 did seem to make it happen less often, but I don't think it's fixed completely.

### Possibilities

Now that I know how to generate Voyant embeds on the fly I can see some exciting possibilities, particularly with things like [Historic Hansard <i class="fa fa-external-link" aria-hidden="true"></i>](http://wragge.github.io/historic-hansard/)!

I'll probably incorporate something like this in the [Closed Access <i class="fa fa-external-link" aria-hidden="true"></i>](http://closedaccess.herokuapp.com/) site. Just need to play and think some more...
