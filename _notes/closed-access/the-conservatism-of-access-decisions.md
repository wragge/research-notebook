---
layout: default_note
note_type: Hacking
title: A way of measuring the conservatism of access decisions?
category: Closed Access
date: 2016-06-19
---

I was wondering about ways of looking for trends in access examination decisions and I had an idea -- what if I looked at the gap between the date the decision was made and the date of the latest content of the file. Generally, you'd assume the older a file is the less likely it is to be closed (note to self -- need to test this). So this gap should provide a sort of proxy measure for the conservatism of the decision. Yes, it's obviously going to vary a lot across reasons, but still...

I decided to start exploring this by looking at the breakdown of decisions for gaps at 10 year intervals -- so find files where the decision was made 10 years or less after the last content was added, and repeat for 20, 30, 40 etc years.

The code is here (and on GitHub):

``` python
def get_age_at_decision(age):
    db = get_db()
    total = 0
    reasons = {}
    for year in range(1966, 2016):
        start_date = datetime.datetime(year, 1, 1)
        end_date = datetime.datetime(year, 12, 31)
        content_date = datetime.datetime(year - int(age), 12, 31)
        # age = decision['date'] - relativedelta(years=int(age))
        total += db.items.find({'access_decision.start_date.date': {'$gte': start_date, '$lte': end_date}, 'contents_dates.end_date.date': {'$gte': datetime.datetime(1800, 12, 31), '$lte': content_date}, 'reasons': {'$ne': ['Parliament Class A']}}).count()
        pipeline = [
            {'$match': {'access_decision.start_date.date': {'$gte': start_date, '$lte': end_date}, 'contents_dates.end_date.date': {'$gte': datetime.datetime(1800, 12, 31), '$lte': content_date}}},
            {'$unwind': '$reasons'},
            {'$group': {'_id': '$reasons', 'total': {'$sum': 1}}},
            {'$project': {'_id': 0, 'reason': '$_id', 'total': '$total'}}
        ]
        results = list(db.items.aggregate(pipeline))
        for reason in results:
            if reason['reason'] != 'Parliament Class A':
                try:
                    reasons[reason['reason']] += reason['total']
                except KeyError:
                    reasons[reason['reason']] = reason['total']
    print 'Age: {}'.format(age)
    print 'Total: {}'.format(total)
    reasons = collections.OrderedDict(sorted(reasons.items()))
    for reason, rtotal in reasons.iteritems():
        print '| {} | {} | {:.2%} |'.format(reason, rtotal, rtotal / float(total))


def get_all_ages():
    for age in [10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110]:
        get_age_at_decision(age)
```

The full results are at the bottom of this page. Actually they're not the full results, because after I ran the script the first time I noticed that the latter age categories were dominated by [Parliament Class A](http://closedaccess.herokuapp.com/reasons/Parliament%20Class%20A/). Parliament Class A records are all closed under regulations attached to the Archives Act, so no decision to close them was actually made -- I'm assuming the 'decision date' is actually just the date they were entered into RecordSearch. Because they're not the result of decisions I decided to exclude them from the data.

The next step was to look at the most conservative decisions per reason -- the files closed for a given reason where the gap between the date of the decision and the date of the content is the greatest.

The code:

``` python
def get_most_conservative(reason):
    db = get_db()
    pipeline = [
        {"$match": {'harvests': harvest_date, 'reasons': reason, 'contents_dates.end_date.date': {'$gt': datetime.datetime(1800, 12, 31)}}},
        {'$project': {'identifier': 1, 'series': 1, 'control_symbol': 1, 'title': 1, 'reasons': 1, 'contents_dates': 1, 'access_decision': 1, 'age': {'$subtract': ['$access_decision.start_date.date', '$contents_dates.end_date.date']}}},
        {'$sort': {'age': -1}},
        {'$limit': 50}
    ]
    items = list(db.items.aggregate(pipeline))
    for item in items:
        age = datetime.timedelta(milliseconds=item['age']).days / 365
        print '{}, {}, {} -- {} years (between {} and {})'.format(item['series'], item['identifier'], item['title'], age, item['access_decision']['start_date']['date'].year, item['contents_dates']['end_date']['date'].year)
``` 

Here's the results for [33(1)(a)](http://closedaccess.herokuapp.com/reasons/33%281%29%28a%29/):

``` text
A11583, 3492833, Imperial Conference 1926 - Memoranda and Papers - E Series - Papers Nos E95 - E140 (With subject index) [3cm] -- 80 years
M2565, 12372904, Personal and official papers of A C RENTOUL [ Mould affected - Reference copy available] -- 73 years
AWM54, 539781, [Chemical Warfare - General:] Allied Land Forces, South West Pacific Area: Technical Notes and Staff Notes - Chemical Warfare, Nov 1942 [Part 3 of 19] -- 70 years
AWM54, 539779, [Chemical Warfare - General:] Allied Land Forces, South West Pacific Area: Technical Notes and Staff Notes - Chemical Warfare, Feb 1942 [Part 1 of 19] -- 70 years
AWM54, 510813, [Chemical Warfare - Trials:] CDRE India: Notes and reports (Oct 1943) [Part 12 of 36] -- 69 years
AWM54, 510812, [Chemical Warfare - Trials:] CDRE India: Notes and reports (Aug 1943) [Part 11 of 36] -- 69 years
AWM54, 510820, [Chemical Warfare - General:] 42nd Chemical Laboratory Company: Technical Reports [Part 7 of 9] -- 69 years
AWM54, 539788, [Chemical Warfare - General:] Allied Land Forces, South West Pacific Area: Technical Notes and Staff Notes - Chemical Warfare, Mar 1943 [Part 10 of 19] -- 69 years
AWM54, 539787, [Chemical Warfare - General:] Allied Land Forces, South West Pacific Area: Technical Notes and Staff Notes - Chemical Warfare, Feb 1943 [Part 9 of 19] -- 69 years
AWM54, 510427, [Chemical Warfare - General:] Deputy Director of Military Operations: Correspondence on the subject of chemical warfare [Part 2 of 2] -- 69 years
AWM54, 543684, [Chemical Warfare - Trials:] Report on Chemical Warfare Physiological Investigations carried out at Townsville, Queensland -- 69 years
AWM54, 510426, [Chemical Warfare - General:] Deputy Director of Military Operations: Correspondence on the subject of chemical warfare [Part 1 of 2] -- 68 years
AWM54, 510804, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 6 of 10] -- 68 years
AWM54, 543763, [Chemical Warfare - Trials:] 1) Revision of basic sampling and venting periods for Chemical Weapons; 2) A review of information derived from trials in the tropics up to May 1944; 3) Reports on trials held in Townsville, Canungra, Holdsworthy, to determine the effect of wearing respirators A/CAS light -- 68 years
AWM54, 510832, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 5 of 10] -- 68 years
AWM54, 510807, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 8 of 10] -- 68 years
AWM54, 510827, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 3 of 10] -- 68 years
AWM54, 510805, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 7 of 10] -- 68 years
AWM54, 510808, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 9 of 10] -- 68 years
AWM54, 510818, [Chemical Warfare - Trials:] CDRE India: Notes and reports (Jan 1944) [Part 15 of 36] -- 68 years
AWM54, 510830, [Chemical Warfare - Trials:] Reports from the Chemical Defence Board, Research and Experimental Section, Innisfail, Queensland [Part 4 of 10] -- 68 years
AWM54, 510838, [Chemical Warfare - Trials:] Reports by 2/1 Australian Chemical Warfare Laboratory, RAE [Part 9 of 12] -- 68 years
AWM54, 510822, [Chemical Warfare - General:] 42nd Chemical Laboratory Company: Technical Reports [Part 8 of 9] -- 68 years
AWM54, 539784, [Chemical Warfare - General:] Australian Military Forces: Technical Notes and Staff Notes - Chemical Warfare, May 1944 [Part 6 of 19] -- 68 years
AWM54, 539785, [Chemical Warfare - General:] Australian Military Forces: Technical Notes and Staff Notes - Chemical Warfare, Jun 1944 [Part 7 of 19] -- 68 years
AWM54, 539777, [Chemical Warfare - General:] Chemical Warfare and Smoke Intelligence Bulletins, prepared by General Headquarters India, General Staff Branch, MI Directorate, Meerut [Part 1 of 7] -- 67 years
AWM54, 663677, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (Mar 1945) [Part 2 of 10] -- 67 years
AWM54, 663686, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (Sep 1945) [Part 8 of 10] -- 67 years
AWM54, 663680, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (May 1945) [part 4 of 10] -- 67 years
AWM54, 663684, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (Aug 1945) [Part 7 of 10] -- 67 years
AWM54, 663672, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (Feb 1945) [Part 1 of 10] -- 67 years
AWM54, 663681, [Chemical Warfare - Trials:] Reports from Australian Field Experimental Station, Proserpine, Queensland (Jul 1945) [Part 6 of 10] -- 67 years
AWM54, 510803, [Chemical Warfare - General:] 42nd Chemical Laboratory Company: Technical Reports [Part 1 of 9] -- 67 years
AWM54, 510802, [Chemical Warfare - General:] Headquarters, United States Army Services of Supply, South West Pacific Area, Office of the Chief Chemical Officer: Chemical Warfare Intelligence Digest, Nov 1944 -- 67 years
AWM54, 539793, [Chemical Warfare - General:] Australian Military Forces: Technical Notes and Staff Notes - Chemical Warfare, Feb 1945 [Part 15 of 19] -- 67 years
AWM54, 1048159, [Chemical Warfare - General:] Chemical Warfare and Smoke Intelligence Bulletins, prepared by General Headquarters India, General Staff Branch, MI Directorate, Meerut [Part 4 of 7] -- 67 years
AWM54, 539778, [Chemical Warfare - General:] Headquarters Army Service Forces, Office of the Chief of Chemical Warfare, Washington 25 DC - Chemical Warfare Intelligence Bulletins, Nos 39-56, Aug 1944 - Jun 1945 -- 67 years
AWM54, 1066679, [Demolitions - Schemes:] Demolition Plans: Brisbane Fortress Area, Belt "C" -- 67 years
AWM54, 1066681, [Demolitions - Schemes:] Demolition Plans: Brisbane Fortress Area, Belt "E" -- 67 years
AWM54, 1066677, [Demolitions - Schemes:] Demolition Plans: Brisbane Fortress Area, Belt "B" -- 67 years
AWM54, 1066680, [Demolitions - Schemes:] Demolition Plans: Brisbane Fortress Area, Belt "D" -- 67 years
A1838, 553313, Antarctic - Heard Island -- 66 years
A5954, 280170, Higher Defence Organisation Constitution and Terms of Reference of Chemical Defence Board ... Chemical and Biological Warfare Subcommittee ... New Weapons and Equipment Committee ... Chemical Defence Board -- 66 years
A5954, 378370, Reports by Secretary Department of Defence on visit abroad 1949. The restoration of the flow of United States classified information to Australia -- 66 years
D1915, 885006, GOODE Mrs AK - delegate to Geneva conference -- 66 years
F504, 756708, Commonwealth Census 1933 (Instructions and Correspondence) Attachments: 1) Census of the Commonwealth of Australia 4th April 1921 Census bulletin No 7 2) An Act to amend the Census and Statistics Act No33 of 1920 3) An Act relating to the Census and Stat6istics of the Commonwealth No15 of 1905 -- 64 years
A816, 510018, Planning for co-operation in British Commonwealth defence (from conclusions of Council of Defence 1950) -- 64 years
A6122, 667099, Indonesian Malay Association - Broome Western Australia -- 63 years
A6119, 943198, Frances Ada GLUCK (aka Garrett, aka Bernie) nee Scott, Volume 3 [125 pages - 115 of these contain exemptions] -- 62 years
A9377, 1014275, [Minutes of Cypher Security Committee, 29 Dec 1943 - 20 Nov 1945] -- 62 years
```

This seemed sort of interesting, so I thought I'd build it into the Closed Access interface. Go to [Most Conservative Decisions?](http://closedaccess.herokuapp.com/mostconservative/) and select a reason to browse files.

[![Closed Access -- most conservative screenshot]({{ site.baseurl }}/images/closedaccess-conservative.png){: .img-responsive .img-example }](http://closedaccess.herokuapp.com/mostconservative/)

----

## Breakdown of reasons by age gap

### Age: 10

**Total: 12402**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 984 | 7.93% |
| 33(1)(b) | 419 | 3.38% |
| 33(1)(c) | 5 | 0.04% |
| 33(1)(d) | 267 | 2.15% |
| 33(1)(e)(i) | 45 | 0.36% |
| 33(1)(e)(ii) | 118 | 0.95% |
| 33(1)(e)(iii) | 21 | 0.17% |
| 33(1)(f)(ii) | 3 | 0.02% |
| 33(1)(f)(iii) | 3 | 0.02% |
| 33(1)(g) | 3770 | 30.40% |
| 33(1)(h) | 5 | 0.04% |
| 33(1)(j) | 19 | 0.15% |
| 33(2)(a) | 9 | 0.07% |
| 33(2)(b) | 9 | 0.07% |
| 33(3)(a)(i) | 29 | 0.23% |
| 33(3)(a)(ii) | 29 | 0.23% |
| 33(3)(b) | 58 | 0.47% |
| Cabinet notebooks | 1 | 0.01% |
| Closed period | 2334 | 18.82% |
| Court records | 4 | 0.03% |
| Destroyed | 1 | 0.01% |
| MAKE YOUR SELECTION | 13 | 0.10% |
| NRF | 2 | 0.02% |
| Non Cwlth-depositor | 3 | 0.02% |
| Non Cwlth-no appeal | 52 | 0.42% |
| Pre Access Recorder | 2630 | 21.21% |
| Withheld pending adv | 3381 | 27.26% |


### Age: 20

**Total: 11260**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 963 | 8.55% |
| 33(1)(b) | 414 | 3.68% |
| 33(1)(c) | 5 | 0.04% |
| 33(1)(d) | 263 | 2.34% |
| 33(1)(e)(i) | 45 | 0.40% |
| 33(1)(e)(ii) | 118 | 1.05% |
| 33(1)(e)(iii) | 21 | 0.19% |
| 33(1)(f)(ii) | 3 | 0.03% |
| 33(1)(f)(iii) | 3 | 0.03% |
| 33(1)(g) | 3442 | 30.57% |
| 33(1)(h) | 5 | 0.04% |
| 33(1)(j) | 18 | 0.16% |
| 33(2)(a) | 9 | 0.08% |
| 33(2)(b) | 9 | 0.08% |
| 33(3)(a)(i) | 28 | 0.25% |
| 33(3)(a)(ii) | 28 | 0.25% |
| 33(3)(b) | 56 | 0.50% |
| Cabinet notebooks | 1 | 0.01% |
| Closed period | 1685 | 14.96% |
| Court records | 4 | 0.04% |
| Destroyed | 1 | 0.01% |
| MAKE YOUR SELECTION | 12 | 0.11% |
| NRF | 2 | 0.02% |
| Non Cwlth-depositor | 3 | 0.03% |
| Non Cwlth-no appeal | 52 | 0.46% |
| Pre Access Recorder | 2281 | 20.26% |
| Withheld pending adv | 3364 | 29.88% |


### Age: 30

**Total: 8166**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 904 | 11.07% |
| 33(1)(b) | 392 | 4.80% |
| 33(1)(c) | 5 | 0.06% |
| 33(1)(d) | 252 | 3.09% |
| 33(1)(e)(i) | 45 | 0.55% |
| 33(1)(e)(ii) | 117 | 1.43% |
| 33(1)(e)(iii) | 20 | 0.24% |
| 33(1)(f)(ii) | 1 | 0.01% |
| 33(1)(f)(iii) | 2 | 0.02% |
| 33(1)(g) | 2808 | 34.39% |
| 33(1)(h) | 5 | 0.06% |
| 33(1)(j) | 10 | 0.12% |
| 33(2)(a) | 8 | 0.10% |
| 33(2)(b) | 8 | 0.10% |
| 33(3)(a)(i) | 25 | 0.31% |
| 33(3)(a)(ii) | 25 | 0.31% |
| 33(3)(b) | 50 | 0.61% |
| Cabinet notebooks | 1 | 0.01% |
| Closed period | 190 | 2.33% |
| Court records | 4 | 0.05% |
| Destroyed | 1 | 0.01% |
| MAKE YOUR SELECTION | 11 | 0.13% |
| NRF | 1 | 0.01% |
| Non Cwlth-depositor | 1 | 0.01% |
| Non Cwlth-no appeal | 32 | 0.39% |
| Pre Access Recorder | 1647 | 20.17% |
| Withheld pending adv | 2819 | 34.52% |


### Age: 40

**Total: 4265**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 387 | 9.07% |
| 33(1)(b) | 197 | 4.62% |
| 33(1)(c) | 1 | 0.02% |
| 33(1)(d) | 126 | 2.95% |
| 33(1)(e)(i) | 28 | 0.66% |
| 33(1)(e)(ii) | 82 | 1.92% |
| 33(1)(e)(iii) | 8 | 0.19% |
| 33(1)(f)(iii) | 2 | 0.05% |
| 33(1)(g) | 1503 | 35.24% |
| 33(1)(h) | 5 | 0.12% |
| 33(1)(j) | 1 | 0.02% |
| 33(2)(a) | 1 | 0.02% |
| 33(2)(b) | 1 | 0.02% |
| 33(3)(a)(i) | 9 | 0.21% |
| 33(3)(a)(ii) | 9 | 0.21% |
| 33(3)(b) | 18 | 0.42% |
| Closed period | 18 | 0.42% |
| Court records | 4 | 0.09% |
| Destroyed | 1 | 0.02% |
| MAKE YOUR SELECTION | 6 | 0.14% |
| NRF | 1 | 0.02% |
| Non Cwlth-depositor | 1 | 0.02% |
| Non Cwlth-no appeal | 2 | 0.05% |
| Pre Access Recorder | 954 | 22.37% |
| Withheld pending adv | 1469 | 34.44% |


### Age: 50

**Total: 2100**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 214 | 10.19% |
| 33(1)(b) | 88 | 4.19% |
| 33(1)(d) | 83 | 3.95% |
| 33(1)(e)(i) | 18 | 0.86% |
| 33(1)(e)(ii) | 18 | 0.86% |
| 33(1)(e)(iii) | 5 | 0.24% |
| 33(1)(f)(iii) | 1 | 0.05% |
| 33(1)(g) | 781 | 37.19% |
| 33(1)(h) | 3 | 0.14% |
| 33(3)(a)(i) | 3 | 0.14% |
| 33(3)(a)(ii) | 3 | 0.14% |
| 33(3)(b) | 6 | 0.29% |
| Closed period | 4 | 0.19% |
| Court records | 2 | 0.10% |
| Non Cwlth-no appeal | 1 | 0.05% |
| Pre Access Recorder | 434 | 20.67% |
| Withheld pending adv | 718 | 34.19% |


### Age: 60

**Total: 747**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 65 | 8.70% |
| 33(1)(b) | 48 | 6.43% |
| 33(1)(d) | 31 | 4.15% |
| 33(1)(e)(i) | 1 | 0.13% |
| 33(1)(e)(ii) | 4 | 0.54% |
| 33(1)(e)(iii) | 1 | 0.13% |
| 33(1)(g) | 318 | 42.57% |
| 33(1)(h) | 1 | 0.13% |
| 33(3)(a)(i) | 1 | 0.13% |
| 33(3)(a)(ii) | 1 | 0.13% |
| 33(3)(b) | 2 | 0.27% |
| Closed period | 2 | 0.27% |
| Non Cwlth-no appeal | 1 | 0.13% |
| Pre Access Recorder | 90 | 12.05% |
| Withheld pending adv | 278 | 37.22% |


### Age: 70

**Total: 225**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 4 | 1.78% |
| 33(1)(b) | 3 | 1.33% |
| 33(1)(d) | 3 | 1.33% |
| 33(1)(g) | 152 | 67.56% |
| 33(1)(h) | 1 | 0.44% |
| Closed period | 1 | 0.44% |
| Pre Access Recorder | 21 | 9.33% |
| Withheld pending adv | 45 | 20.00% |


### Age: 80

**Total: 27**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(a) | 1 | 3.70% |
| 33(1)(d) | 1 | 3.70% |
| 33(1)(g) | 13 | 48.15% |
| 33(1)(h) | 1 | 3.70% |
| Pre Access Recorder | 9 | 33.33% |
| Withheld pending adv | 2 | 7.41% |


### Age: 90

**Total: 6**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(g) | 3 | 50.00% |
| Pre Access Recorder | 2 | 33.33% |
| Withheld pending adv | 1 | 16.67% |


### Age: 100

**Total: 2**

{: .table}
| Reason | Number | Percentage of total |
|----|----|----|
| 33(1)(g) | 2 | 100.00% |