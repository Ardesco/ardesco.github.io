---
layout: post
author: "Mark Collin"
title: "Automated Tests Should Be Living Documentation"
date: 2014-09-17 06:46:00 +0100
categories: [Testing] 
tags: [Thoughts]
---
To start off let me clarify that when I talk about an automated test in the title, I mean an automated check.  I’m aware of the difference, but most people I interact with on a daily basis still talk about automated tests, not automated checks.

I often see people talking about adding automated checks to “check that something still works”.  I think that if you are doing this, you are looking at your automated checks in the wrong way.

## The Ideal

TDD is a methodology that is prevalent in the modern development landscape and the base idea is that you write some failing checks in advance (writing these checks first helps to highlight bad design patterns) and then you write the required code to make them pass.  These checks were never really designed to “check” that something still works, they are describing a series of desired behaviours that you want your program to exhibit and then the code is written to make program work in that way.

Now to me this sounds like documentation, we have a series of desired behaviours that have been expressed in code that tell the developer (or the tester, or anybody that cares to look at them) how the program works.

This documentation may not be perfect, it may be hard to read (Think of a cheap electrical product you have bought that had a manual In Chinese instead of English), it may be severely limited (are there more checks you should add?) and it some cases it may even be wrong (We have all seen automated checks that never fail).  These are all very real issues, but they are issues that are not insurmountable obstacles.

The important thing to note is that as the program changes, these automated checks change as well, they evolve with the product and continue to explain how it works in its current state, just like any halfway decent set of documentation.

How do we achieve this?  Well we make sure that the development team (and I do mean team, not just the developers in the team, or the testers in the team) are responsible for the automated checks and ensure that they are added appropriately and that they run all of them to make sure that they pass before pushing code to the central code repository.

This gives us a high level of confidence that things work in the way we want them to work, and a fast feedback loop if they don’t.  It also means that our documentation (the automated checks) are always up to date because it is changed as the code changes.

## The Sad Reality…

So what is the point of this post?  Well I regularly see people who are not part of the development team writing automated checks after the fact, your classic “Test Automation Team”.  The focus of these teams is generally to write a series of checks that ensure the program still works in an expected way, in other words checks that detect change (you may be used to the term “regression tests”).

The problem with this is that while your developers are working on a product, it’s constantly changing, so the checks that the automation team have written will constantly be detecting this change (or to put it another way, checks will constantly be failing).  In this scenario automated checks are not seen as documentation, they are seen as a validation layer and this (in my mind) is a bad thing!

When there is a failure in the validation layer defects are raised, time is taken away from the development team to triage non-issues and the automated test team are slowly getting further and further behind as they struggle to keep up with all the changes that are being made as well as adding new checks for new functionality.

While all of this is happening the checks that are being run by the test automation team are rarely ever green because new changes keep coming in that break the test automation build and we all knows what this means, nobody trusts them…

>“Oh look the test automation team’s board is red again”
>
>“Don’t worry about it, they probably haven’t updated their checks to deal with our latest change…”

People start to distrust the automated checks written by the test automation team, and this then normally results in the test automation team announcing that they need more resource to be able to keep up with the workload, however this then just exacerbates the problem.

You’re now in a downward spiral of test automation hell, where we are writing more and more tests, things keep failing for no good reason and problems start slipping through the cracks.  People get stressed, the feedback loop is getting longer and longer and things keep breaking.

Sound like any projects you have worked on?