# Overview

Purpose:  Analyze a live zoom call : live transcript with speaker separation.  Text analytics. 
Fast search, playback and clipping of snippets of a call for sharing.

## Details

**Cognitive Category**
NLP

**Status**
TODO


**User Flow**

1. User can connect the app to as many Google calendars as they want
1. Log of all analyzed calls, including the status to show live calls.
1. Real-time call analysis should have sub 5 second lag (2 would be preferable).

Actions:
1. User can click on a call to bring up the analysis dashboard (~ Illuminate)
1. User can direct click the link in the calendar:  www.ZOOMAnalysis.com?callguid=[CALL GUID]
1. Hooks into Redact, Library, Collections, Share, etc..

Automate Flows under the covers:
1. When a user connects a calendar - Call Job #1
1. When a user disconnects a calendar - Flow Not Shown
