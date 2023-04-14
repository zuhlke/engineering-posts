---
title: Layers of a Code Review
subtitle: The different layers of depth you go through while reviewing code
domain: software-engineering-corner.hashnode.dev
tags: development, code-review, communication
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1680181506757/dqCCyfD4E.jpg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
hideFromHashnodeCommunity: false
publishAs: culas
ignorePost: true
---

In a recent discussion about how we approach and conduct code reviews (most often during a pull request), we came to question which aspects of the code under review one is looking at.
One comment from that discussion stuck especially with me.
Although it hasn't changed much in terms of how I review code, it changed the way I think about code reviews and approach them more methodically(?).
Reviewing code is digging through layers of depth.
Come with me on a journey through those layers I identified.

## Surface

- partially automatic (code style/Prettier, linting)
- debatable, personal taste/style, can lead to "bike shedding"
- low impact on product quality/reliability
- some impact on maintainability/readability
- easy to detect, judge, comment on and fix
- little gain upon improvement

## Functional

- questions: does it work?
- looking at function bodies, self-contained sections

## Design

- architecture, design, structure
- data flow
- maintainability
- effects on codebase

open question: Functional or Design first?

## Core

- questions: does it do the right things?
- fulfilling requirements from story/issue?

## Summary

As we go through changed code, we start at the surface level, dig deeper into how the different pieces play together to eventually be able to judge whether it does the right things.
The importance of a comment depends mostly on the level they relate to.

Having these layers in the back of your mind helps you not skipping parts.
You might even catch yourself not going beyond the surface layer.
Also use these layers to talk about code reviews in your team.
