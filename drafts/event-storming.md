---
title: DDD: How to do Event-Storming?
domain: campzulu.hashnode.dev
tags: software-architecture, ddd
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1670243050541/SwGJDMFY4.png?auto=compress
publishAs: keerthikan
ignorePost: true
---

# DDD: How to do Event Storming?

At Zühlke, there is a **DDD Topic Team** to discuss about various topics related to Domain-Driven-Design. Recently, we had a session about **Event Storming**. In this post, I would like to share how we did it and some key takeaways.

## What is Event Storming?

Event Storming is a workshop-based method to collect all possible events that can happen in the system. This workshop takes place as one of the early meeting where software developers and domain experts bring in as much ideas/events as possible from their domain knowledge. This facilitates the participants to learn from each other and get the big picture of the problem domain.

The result of the meeting is sticky notes spread around in a feasible timeline. This artifact can then be used as a means for requirement engineering.

## The Procedure

To practise during the DDD Topic Team meeting, we decided to do event storming for the time period between the marriage proposal💍 and the wedding 👰🤵. It was not a real work example for desigining a software system, however it was good to understand the method outside software development.

**1. Collect domain events individually (~10min)**

As first, we started to brainstorm individually for about 10 minutes and collected possible domain events. An event is written in the passive form (e.g. *Invitations are sent*) and denoted in an orange sticky as shown below.

<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1670406732876/FXyyefueU.png?auto=compress" alt="domain-event" width="100"/>

The collected events can be chaotic and redundant on the board. That is okay. The important benefit of this step is that it gives everyone the chance to think off and collect events from their perspective.

**2. Pick some pivotal events (~5min)**

As next step, we tried to pick few pivotal events together. A pivotal event means it is a very significant event in the process. They are very helpful to initially set the timeline. An example of the initial timeline with few pivotal events would look as shown below.

![Pivotal Events](https://cdn.hashnode.com/res/hashnode/image/upload/v1670414430346/KeLQdHR4t.png?auto=compress)

| *Source: https://leanpub.com/ddd_first_15_years – Discovering Bounded Contexts with EventStorming — Alberto Brandolini*

Tip: Don't spend too much time on discussing if it is a pivotal event or not. The main purpose of this step is to have a widespread timeline as a baseline.

**3. Order the events in a timeline (~30min)**

In the next step, we started to order the remaining domain events in-between the pivotal events. Thereby, we enforce a timeline and remove any duplicates. If two events could happen in any order (also known as `causally unrelated`), put the sticky notes in parallel.

During the discussion, a domain event could trigger some questions, problems or conflicts. It is well worth to capture them as shown below and continue with the ordering.

<img src="https://cdn.hashnode.com/res/hashnode/image/upload/v1670417329757/gShyOCEmz.png?auto=compress" alt="hot spots" width="150"/>

**4. Recap (~5min)**

Once the events were ordered in a timeline, we recaped the session by briefly telling a story based on the events from left to right. This short summary gave us a good overview of the process flow.

The final output of the session looks as shown below. 

![Event Storming](https://cdn.hashnode.com/res/hashnode/image/upload/v1670243304928/PlghMRLDK.png?auto=compress)

## Next Steps



## Key Takeaways


## Helpful Resources
* [Event Storming Glossary & Cheet Sheet](https://github.com/ddd-crew/eventstorming-glossary-cheat-sheet)
