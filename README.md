Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Secretary of State for Education](https://www.wikidata.org/wiki/Q3477306)
contains all the data expected already.

Step 2: Tracking page
=====================

PositionHolderHistory already exists; current version is
https://www.wikidata.org/w/index.php?title=Talk:Q3477306&oldid=1237559970
with 33 dated memberships and 22 undated; and 54 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = [Secretary of State for Education](https://en.wikipedia.org/wiki/Secretary_of_State_for_Education)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Needed a bit of a tweak to cope with the nested table where there were
sepearate Secretaries of State for Children, Schools and Families; and
Innovation, Universities and Skills.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

25 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/a9bb3870d3f0e/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

92 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/623924359a4e1/

I also manually corrected David Blunkett to Shadow, and Sarah Teather to
Lib Dem spokesperson.

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q3477306&oldid=1237592658

We have 4 higher-precision dates that can be accepted, and one good
correction:

    Q6114554$BC3DA225-A615-4A15-8465-DEF389D871DD P580 1911-01-01 1911-10-23
    Q6114554$BC3DA225-A615-4A15-8465-DEF389D871DD P582 1915-01-01 1915-05-25
    Q336421$B373EF5C-3C04-4DB2-9D09-B57F1792B961 P580 1916-01-01 1916-08-18
    Q336421$B373EF5C-3C04-4DB2-9D09-B57F1792B961 P582 1916-12-01 1916-12-10
    Q333694$2CCF8E85-2C11-4E8D-AB7E-E6E02A53E33B P580 1981-09-11 1981-09-14

It also looks like there's a potentially legitimate overlap around the
turn of the 19th-20th century between the President of the Board of
Education and Vice-President of the Committee of the Council on
Education.

Final version: https://www.wikidata.org/w/index.php?title=Talk:Q3477306&oldid=1237637999
