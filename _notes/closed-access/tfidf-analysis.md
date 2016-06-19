---
layout: default_note
note_type: Hacking
title: TF-IDF analysis
category: Closed Access
date: 2016-06-10
---

I'm trying out [scikit-learn](http://scikit-learn.org/stable/) for calculating TF-IDF values for words and phrases in file titles. I'm basically copying this [nice tutorial](http://www.markhneedham.com/blog/2015/02/15/pythonscikit-learn-calculating-tfidf-on-how-i-met-your-mother-transcripts/) by Mark Needham.

I've [already generated]({{ site.baseurl }}/notes/analysing-file-titles-i/) a file containing the titles for each reason. I'm just going to feed all these files to scikit-learn's TfidfVectorizer and then list the top 10 phrases for each reason.

This is the code:

``` python
import os
from sklearn.feature_extraction.text import TfidfVectorizer


data_dir = os.path.join(os.getcwd(), 'data')
# Get a list of the reasons from the file titles
names = [file[7:len(file) - 12] for file in os.listdir(data_dir) if file[-4:] == '.txt']
# Get a list of filenames to feed to scikit-learn
files = [os.path.join(data_dir, file) for file in os.listdir(data_dir) if file[-4:] == '.txt']
# Chomp chomp -- getting trigrams
tf = TfidfVectorizer(input='filename', analyzer='word', ngram_range=(3, 3), min_df=0, stop_words='english')
tfidf_matrix = tf.fit_transform(files)
# These are the actual phrases
feature_names = tf.get_feature_names()
# These are the scores
reasons = tfidf_matrix.todense()
for index, row in enumerate(reasons):
    name = names[index]
    print '\n\n{}\n'.format(name.upper())
    reason = row.tolist()[0]
    # If the score is not 0 save it with an index (which will let us get the feature_name)
    scores = [pair for pair in zip(range(0, len(reason)), reason) if pair[1] > 0]
    sorted_scores = sorted(scores, key=lambda t: t[1] * -1)
    # Print the top 10 results
    for phrase, score in [(feature_names[word_id], score) for (word_id, score) in sorted_scores][:10]:
        print('{0: <40} {1}'.format(phrase, score))
```

And here's the results!

```
33-1-A

communist party australia                0.49873257731
branch communist party                   0.259836066422
chemical defence board                   0.137982195947
brisbane fortress area                   0.122176195446
fortress area belt                       0.122176195446
abstract reports indexed                 0.120770326619
indexed records chemical                 0.120770326619
laboratories abstract reports            0.120770326619
records chemical defence                 0.120770326619
reports indexed records                  0.120770326619


33-1-B

chemical defence board                   0.226129044108
abstract reports indexed                 0.212582626787
indexed records chemical                 0.212582626787
laboratories abstract reports            0.212582626787
records chemical defence                 0.212582626787
reports indexed records                  0.212582626787
supply laboratories abstract             0.212582626787
munitions supply laboratories            0.203566533876
board munitions supply                   0.191324364108
defence board munitions                  0.191324364108


33-1-C

1971 continental shelf                   0.167605323718
australia 1971 continental               0.167605323718
drainage storage vaults                  0.167605323718
file lc235 law                           0.167605323718
guinea reserve bank                      0.167605323718
lc235 law sea                            0.167605323718
stormwater drainage storage              0.167605323718
1978 decision 5230                       0.149109035956
328 minister responsible                 0.149109035956
5230 ad hoc                              0.149109035956


33-1-D

census 1933 district                     0.438168509987
communist party australia                0.38446057734
branch communist party                   0.269095324415
party australia nsw                      0.161593522809
party australia branches                 0.149497402453
hospital outpatient cards                0.113115465966
woomera hospital outpatient              0.113115465966
cards woomera hospital                   0.0969561136852
outpatient cards woomera                 0.0969561136852
australia branches victoria              0.0896984414716


33-1-E-I

rowland keren ellen                      0.369543809895
volume johnston elliott                  0.151823573857
ellen dissapearance death                0.123181269965
ellen missing person                     0.123181269965
johnston elliott frank                   0.123181269965
keren ellen dissapearance                0.123181269965
keren ellen inquest                      0.123181269965
keren ellen missing                      0.123181269965
austral bronze branch                    0.11396050764
ernest archer russell                    0.11396050764


33-1-E-II

rowland keren ellen                      0.178869481818
grange branch cpa                        0.0784525250819
ellen dissapearance death                0.0715477927273
keren ellen dissapearance                0.0715477927273
keren ellen inquest                      0.0715477927273
pakistan paper contributed               0.0715477927273
paper contributed pakistan               0.0715477927273
threat pakistan paper                    0.0715477927273
action file non                          0.0661920662291
ada gluck aka                            0.0661920662291


33-1-E-III

case january 1975                        0.475505736598
1975 case january                        0.260697250782
reserve bank australia                   0.237752868299
january 1975 case                        0.173798167188
17919 submission case                    0.0976785138688
1973 asis station                        0.0976785138688
1975 heiser ronald                       0.0976785138688
1975 production october                  0.0976785138688
1975 reserve bank                        0.0976785138688
1975 romanov nicolai                     0.0976785138688


33-1-F-II

assessment heroin importations           0.287161534805
chinese crew member                      0.287161534805
commonwealth police victoria             0.287161534805
crew member protective                   0.287161534805
heroin importations chinese              0.287161534805
importations chinese crew                0.287161534805
intelligence assessment heroin           0.287161534805
member protective security               0.287161534805
police victoria police                   0.287161534805
review commonwealth police               0.287161534805


33-1-F-III

character checking victoria              0.314601078767
checking victoria police                 0.314601078767
police records anti                      0.314601078767
records anti sabotage                    0.314601078767
22 39 39                                 0.279882897064
anti sabotage vulnerable                 0.279882897064
croatian character checking              0.279882897064
points 22 39                             0.279882897064
sabotage vulnerable points               0.279882897064
vulnerable points 22                     0.279882897064


33-1-G

aboriginal people torres                 0.349683210861
people torres strait                     0.349683210861
strait islander peoples                  0.349683210861
torres strait islander                   0.329537265927
ran medical officers                     0.224605599632
islander peoples tribal                  0.177812766376
medical officers journal                 0.149737066422
box personal index                       0.114174513146
index cards titled                       0.114174513146
personal index cards                     0.114174513146


33-1-H

application registration design          0.301958605139
class 4a application                     0.226276784365
darbyshire pottery pty                   0.226276784365
design darbyshire pottery                0.226276784365
pottery pty small                        0.226276784365
registration design darbyshire           0.226276784365
4a application letters                   0.113138392183
4a application registration              0.113138392183
aboriginal class 4a                      0.113138392183
application letters patent               0.113138392183


33-1-J

officers ran personal                    0.187798132754
ran personal record                      0.187798132754
11 february 1987                         0.166795698317
31st meeting council                     0.166795698317
462 minutes 31st                         0.166795698317
attorney general canberra                0.166795698317
australian war memorial                  0.166795698317
canberra suspected collusive             0.166795698317
council 11 february                      0.166795698317
general canberra suspected               0.166795698317


33-2-A

submerged lands advice                   0.233847237862
cabinet minute decision                  0.21634254903
seas submerged lands                     0.21634254903
submission cabinet paper                 0.21634254903
1810 1848 seas                           0.128207353396
1848 seas submerged                      0.128207353396
1967 cabinet minute                      0.128207353396
act 1967 cabinet                         0.128207353396
adjacent australia resources             0.128207353396
advice joint opinion                     0.128207353396


33-2-B

submerged lands advice                   0.233847237862
cabinet minute decision                  0.21634254903
seas submerged lands                     0.21634254903
submission cabinet paper                 0.21634254903
1810 1848 seas                           0.128207353396
1848 seas submerged                      0.128207353396
1967 cabinet minute                      0.128207353396
act 1967 cabinet                         0.128207353396
adjacent australia resources             0.128207353396
advice joint opinion                     0.128207353396


33-3-A-I

beneficiaries general file               0.305788314863
estates beneficiaries general            0.305788314863
trust estates beneficiaries              0.305788314863
file trust estates                       0.203858876575
general file trust                       0.203858876575
legal tax avoidance                      0.203858876575
avoidance legal tax                      0.152894157431
butt geoffrey percival                   0.152894157431
creation inter vivos                     0.152894157431
geoffrey percival trustees               0.152894157431


33-3-A-II

beneficiaries general file               0.305788314863
estates beneficiaries general            0.305788314863
trust estates beneficiaries              0.305788314863
file trust estates                       0.203858876575
general file trust                       0.203858876575
legal tax avoidance                      0.203858876575
avoidance legal tax                      0.152894157431
butt geoffrey percival                   0.152894157431
creation inter vivos                     0.152894157431
geoffrey percival trustees               0.152894157431


33-3-B

beneficiaries general file               0.305788314863
estates beneficiaries general            0.305788314863
trust estates beneficiaries              0.305788314863
file trust estates                       0.203858876575
general file trust                       0.203858876575
legal tax avoidance                      0.203858876575
avoidance legal tax                      0.152894157431
butt geoffrey percival                   0.152894157431
creation inter vivos                     0.152894157431
geoffrey percival trustees               0.152894157431


CABINET-NOTEBOOKS

1973 special annex                       0.353553390593
australia japan ministerial              0.353553390593
committee meeting tokyo                  0.353553390593
japan ministerial committee              0.353553390593
meeting tokyo 1973                       0.353553390593
ministerial committee meeting            0.353553390593
special annex sa                         0.353553390593
tokyo 1973 special                       0.353553390593


CLOSED-PERIOD

aboriginal islander community            0.120253579073
community health service                 0.120253579073
islander community health                0.120253579073
images colour 35mm                       0.102215542212
boer war dossier                         0.0901901843051
james service number                     0.0855862801878
1983 photographic negative               0.0841775053514
milton order australia                   0.0841775053514
order australia medal                    0.0841775053514
stevens milton order                     0.0841775053514


COURT-RECORDS

company limited versus                   0.342997170285
ankan shipping company                   0.171498585143
buaja coles company                      0.171498585143
china navigation company                 0.171498585143
china ocean shipping                     0.171498585143
coles company limited                    0.171498585143
commissioner taxation versus             0.171498585143
company ex parte                         0.171498585143
company limited china                    0.171498585143
corporation ankan shipping               0.171498585143


DESTROYED

treatment patients timor                 1.0


MAKE-YOUR-SELECTION

1967 democratic peoples                  0.12480047223
400075 seas submerged                    0.12480047223
8023 400075 seas                         0.12480047223
955 lrmp project                         0.12480047223
act 1967 democratic                      0.12480047223
albert ernest fletcher                   0.12480047223
australia internal security              0.12480047223
correspondent royal commission           0.12480047223
coup murdoch robert                      0.12480047223
developments extremist terrorist         0.12480047223


NON-CWLTH-DEPOSITOR

1969 election policy                     0.258198889747
eggleton press secretary                 0.258198889747
election policy speech                   0.258198889747
files maintained eggleton                0.258198889747
maintained eggleton press                0.258198889747
minister 1969 election                   0.258198889747
nicholls society subject                 0.258198889747
policy speech policy                     0.258198889747
policy speech reaction                   0.258198889747
press secretary prime                    0.258198889747


NON-CWLTH-NO-APPEAL

gibbs chief justice                      0.410693453107
judge notes gibbs                        0.365060847206
notes gibbs chief                        0.365060847206
high court memoranda                     0.273795635405
chief justice judge                      0.136897817702
justice judge notes                      0.136897817702
1983 judge notes                         0.0912652118016
1983 personal correspondence             0.0912652118016
chief justice high                       0.0912652118016
correspondence justices high             0.0912652118016


NRF

aboriginal arrangements mental           0.353553390593
aboriginal policy procedures             0.353553390593
arrangements mental patients             0.353553390593
defence tindal raaf                      0.353553390593
mental patients aboriginal               0.353553390593
patients aboriginal policy               0.353553390593
raaf aboriginal arrangements             0.353553390593
tindal raaf aboriginal                   0.353553390593


PARLIAMENT-CLASS-A

national security act                    0.40439158801
act regulations ordinances               0.40095288403
security act regulations                 0.40095288403
ordinances national security             0.361751658662
regulations ordinances national          0.361751658662
national security general                0.232500845569
security general regulations             0.226994246595
general regulations reg                  0.141674603963
submissions briefing papers              0.0894063034716
59 national security                     0.0804656731244


PRE-ACCESS-RECORDER

new guinea staff                         0.372630672985
royal commission petroleum               0.213580995491
staff permanent central                  0.199948165992
tax assessment act                       0.168138230493
periodical operational awards            0.136328294994
permanent central victoria               0.131784018495
income tax assessment                    0.122695465495
assessment act 1922                      0.104518359496
immediate periodical operational         0.104518359496
grade diver staff                        0.0954298064961


WITHHELD-PENDING-ADV

china relations australia                0.27381460646
indo chinese refugees                    0.244825035571
south east asia                          0.170567684071
australian policy indo                   0.167880024392
papua new guinea                         0.14435301024
economic relations australia             0.10890353666
united states america                    0.102971737069
chinese refugees australian              0.101427514737
policy indo chinese                      0.101427514737
china australian policy                  0.0944325137204
```