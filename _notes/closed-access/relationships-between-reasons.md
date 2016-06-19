---
layout: default_note
note_type: Hacking
title: Relationships between reasons
category: Closed Access
date: 2016-06-19
---

In many cases there'll be multiple reasons cited for closing a file. I thought I might try a simple network visualisation using Gephi to get a sense of which reasons occurred together.

To do this I needed to create two CSV files listing:

* the reasons (the nodes of the graph)
* pairs of reasons that are cited together (the edges of the graph)

Reasons cited for each file are stored as an array. So to create the edges I just looped through this array, writing out each combination of reasons. The direction of the relationship doesn't matter, so all the edges are labelled 'undirected'.

The code is here (and [on GitHub](https://github.com/wragge/closed_access/blob/master/relationships.py)):

``` python

import csv
from pymongo import MongoClient
from credentials import MONGOLAB_URL
import datetime

harvest_date = datetime.datetime(2016, 1, 1)


def get_db():
    dbclient = MongoClient(MONGOLAB_URL)
    db = dbclient.get_default_database()
    return db


def get_reasons():
    db = get_db()
    agg = db.aggregates.find_one({'harvest_date': harvest_date, 'agg_type': 'reason_totals'})
    reasons = [reason['reason'] for reason in agg['results']]
    return reasons


def get_edges(source, targets):
    edges = []
    for target in targets:
        edges.append([source, target, 'Undirected'])
    return edges


def make_edges():
    db = get_db()
    reasons = get_reasons()
    for item in db.items.find({'harvests': harvest_date}):
        edges = []
        nodes = [reasons.index(reason) for reason in item['reasons']]
        while len(nodes) > 1:
            source = nodes[0]
            nodes.pop(0)
            edges += get_edges(source, nodes)
        if edges:
            with open('edges.csv', 'ab') as edges_csv:
                csv_writer = csv.writer(edges_csv)
                for edge in edges:
                    csv_writer.writerow(edge)


def make_nodes():
    reasons = get_reasons()
    with open('nodes.csv', 'wb') as nodes_csv:
        csv_writer = csv.writer(nodes_csv)
        for index, reason in enumerate(reasons):
            csv_writer.writerow([index, reason])

```

Running `make_edges()` and `make_nodes()` generates the two CSV files formatted according to Gephi's requirements.

* [nodes.csv](https://github.com/wragge/closed_access/blob/master/data/nodes.csv)
* [edges.csv](https://github.com/wragge/closed_access/blob/master/data/edges.csv)

I'm a bit of a Gephi newbie, so I basically just followed [Martin Grandjean's tutorial](http://www.martingrandjean.ch/gephi-introduction/) to load my data. I fiddled around with the options a bit and ended up with a graph where the size of the nodes represents their `degree` (the number of connections) and the colour represents their `betweenness centrality`.

[![Network visualisation]({{ site.baseurl }}/images/reasons_graph_labelled.png){: .img-responsive .img-example}]({{ site.baseurl }}/images/reasons_graph_labelled.png)

It's no surprise that 33(1)(a) and 33(1)(g) figure prominently as they're the most often cited exemptions defined in the Archives Act. But it's interesting that 33(1)(a) has almost as many connections as 33(1)(g), when the latter is cited about four times as much. The centrality of 33(1)(a) reflects the fact that it's used in conjunction with a range of other reasons. My assumption is that 33(1)(a) is the go-to exemption for anything security related, with other, more specific, exemptions added as required. 

The centrality and rate of citation of 33(1)(a) and 33(1)(g) also suggests that the range of available reasons could be simplified.




