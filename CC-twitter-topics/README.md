# Overview

Title: Twitter Classifier
Description: This flow engine scrapes the tweets of a provided Twitter user and processes each tweet to create topics associated with each tweet.

## Details

**Cognitive Category**

Content Classification

**Status**

TODO

**Notes**

1. An empty TemporalDataObject (TDO) is created for each run of the Flow engine
1. Each tweet is written to a Structured Data Object (SDO) in aiWARE
1. At the same time, each tweet's content is processed by a topic extraction engine (eg, ZettaCloud)
1. Each result is written back to the SDO, and the SDOs are correlated to the TDO created at start time
