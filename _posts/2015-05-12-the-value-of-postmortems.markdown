---
layout: post
title:  "The Value of Postmortems"
author: Patrick Hereford
github_username: phereford
date:   2015-05-12 23:15:00
categories: culture engineering
---

Things go wrong, especially in software. One of the more frustrating elements of
software engineering is troubleshooting off hour issues that seemingly happen
randomly (i.e. no software changes on the system in question). Sometimes it's
how you have integrated into a 3rd party vendor. Other times the interent as a 
whole is down due to some widespread DDOS attack. No matter the reason, I
truly believe that every company should have a policy of creating Postmortems
for internal and possibly external use.

## Difficulties
The largest obstable to overcome is shifting the point of view of downtime from
blame to being completely [blameless](https://codeascraft.com/2012/05/22/blameless-postmortems/). This shift _has to happen_ for postmortems to have any value to
an organization. The main point of postmortems is to disseminate information and
educate team members for specific issues that bring down systems.

If blame is involved, it deincentivizes the entire structure of a postmortem and
silos system knowledge into the few that tackle those issues.

## Benefits
At the very least, every engineer should write a detailed "report" about the
downtime that took place.  Screenshots are encouraged (we use a mix of NewRelic
and Librato for pretty graphs). At America's Test Kitchen, we document the
following:  
- Summary (what happened, symptoms)  
- How did the issue come to our attention?  
- Impact  
- Root cause  
- Short-term fix  
- Long-term fix  
- Related JIRA issue links  

The main benefit from a postmortem policy is information dissemination. Every
engineer can be made aware of a set of symptoms for specific issues and make a
determination as to the best course of action. In addition, postmortems can also
serve as a historic basis to determine severity of issues and frequency of
issues.

## Cons
There is a lot of overhead in writing postmortems, especially at 4am. An
engineer has to be very mindful that they will be writing a postmortem after
the downtime such that they create a repeatable trace of commands,
analysis of logs, etc. Writing a postmortem well after the events have
transpired with little documentation can lead to incorrect diagnosis and
misremembering the various analysis to determine the root cause of the issue.

It's paramount that the postmortem be written no later than 48 hours after the
downtime. Analysis and command history get lost quickly, especially in the heat
of the moment.

## Conclusion
Any organization that is technical in nature should implement some form of 
documentation about downtime. While the creation of these documents is pure
overhead, it will end up paying for itself in spades as the  symptoms and
solutions get disseminated to the engineering team as a whole.
