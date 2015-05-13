---
layout: post
title:  "The Value of Postmortems"
author: Patrick Hereford
github_username: phereford
date:   2015-05-13 13:15:00
categories: culture engineering
---

It's 2am and your phone wakes you. Servers down. Sleepily, you muster the energy
to get out of bed and start troubleshooting system issues and resolve them after
two hours. You are able to go back to sleep for that extra beauty rest
before going into work. Of course the moment you are in the office, everyone
wants to know of your heroic deeds, from the steps needed to resolve the issue
to the actual issue at hand. You painstakingly recount the root issue and the
attempt to discuss the exact steps taken to solve it. Everyone is intently
listening to your story when all of a sudden, you forget the order of operations
needed to resolve the issue.

Whether or not you forget the replication steps to solve the issue, this 
rather familiar scenario is where a postmortem can help your team. A postmortem
would immediately disseminate the necessary information (asynchronously)
reducing the need for this somewhat awkward group discussion. In addition,
the engineer should be receiving [the highest of fives](http://media.giphy.com/media/Cp9cUJc4hgNvG/giphy.gif)
for getting the business back online _AND_ appropriately documenting every
step of the process.
  
---
  
## Shifting to a Blameless Culture
When considering postmortems for your company, one has to be mindful of
maintaing a culture of being completely [blameless](https://codeascraft.com/2012/05/22/blameless-postmortems/).
This _has to exist_ for postmortems to have any value to an organization.
The main point of postmortems is to disseminate information and educate team 
members on specific issues that bring down systems. We want to break up those
important system wide knowledge silos in as positive a fashion as we can.
  
Let's say Greg had been working rather late one evening to ship a feature. He had
accidentally left out the all important `:` in our `clock.rb` file, causing all
of our scheduled jobs to fail. This failure actually went undetected for an
extra day or two as our error provider never picked up any errors (our clock
process would not boot up).
  
More often than not, Greg will more than likely already feel terrible for being
the root contributor to the issue at hand. Adding a layer of blame to that is
just not needed or warranted and may have negative effects on his professional
development.
  
For the sake of our postmortem, it also does not matter one bit that Greg was at 
fault for this issue. What matters is that the symptoms of the issue are 
correctly documented, the solution is well documented, and there are regression
tests for this scenario (when applicable).

## Tools We Use
We really enjoy using [discourse](https://github.com/discourse/discourse) for
documenting everything that relates to our development processes, including
our postmortems. Not only does it enable a team of 
developers to document all the things using the tools we know and love
(i.e. Markdown), but the threaded forum format _encourages continued discussion_ 
of the issues involved. The search quality is pretty good, too.
  
## Overcoming Hurdles
4am isn't the easiest time to write eloquent prose. As engineers, we need to be in 
the habit of taking notes and snapshots as we go (repeatable trace of commands,
analysis of logs, etc), being mindful of the need to compile a postmortem afterwards.
Writing a postmortem well after the events have
transpired with little documentation can lead to incorrect diagnosis and
misremembering the various analyses performed to determine the root cause of the
issue.  It's paramount that the postmortem be written no later than 48 hours after the
downtime. Analysis and command history get lost quickly, especially in the heat
of the moment.

Any organization that is technical in nature should implement some form of 
documentation about downtime and critical issues. The thoughtful curation and 
creation of these long lived documents will help any organization in the long
term through knowledge dissemination.

___

[Discuss on HN]()
