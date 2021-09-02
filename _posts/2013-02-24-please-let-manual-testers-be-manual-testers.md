---
layout: post
author: "Mark Collin"
title: "Please Let Manual Testers Be Manual Testers"
image: /assets/images/feed/tester.jpg
attribution: "Photo by JESHOOTS.COM on Unsplash"
category: Testing
tags: [Manual Testing]
---
The testing world seems to have entered a state of flux in the last couple of years where “Automated Testing” is the new nirvana, I suspect this in part due to more and more companies following Google’s lead and starting to hire developers in test. Now in my opinion having people performing these roles is not a bad thing. When you have somebody writing your test framework you want somebody with some experience writing code and making architectural decisions. A developer in test is a very useful and powerful resource in this situation. The problem is that people have seen how useful a developer in test is, and they have decided that every tester should now become a developer in test, even though the majority of testers are probably not going to be performing automated testing, or writing test frameworks.

I regularly frequent the Selenium User's mailing list and day by day, I see more and more people coming to the list who just don’t seem to have a basic clue about programming. These people invariably want you to ‘urgently’ help them because they have to write an automated test/test framework and they have no idea how to do it. Now people wanting to learn Selenium and become automated testers is not a bad thing, but most of these requests seem to be people who have testing jobs and have suddenly had the job of automated tester thrust upon them, this is a bad thing!

The other thing I see more and more regularly now is test frameworks that have been written so that manual testers can easily use them without learning how to program. These products seem to come in two forms:

- Something that scans a page finding all the elements of interest to abstract away the logic of finding them.Let manual testers be manual testers
- An Excel spread sheet driven framework where you have to manually populate the spread sheet with locators/expected text to run the tests.

Now I can see some value in option 1, that could be useful for automated testers who don’t want to spend lots of time locating things that may be interesting on a page and just want to spend time interacting with elements. Personally however I would not want to use something like this, I prefer to locate elements of interest myself to keep my tests lean and mean.

Option 2 is something that should be killed right now in my opinion. It has exclusively been written to take manual testers and trick them with the promise that they will now become automated testers and eventually developers in test. It will not do this; they will spend all of their day filling in Excel spread sheets (Why do we still have such an obsession with Excel spread sheets anyway?) and then the rest of their time updating them as things go wrong.

This process really turns manual testers into data entry clerks. These frameworks are invariably brittle as hell and require a lot of manual effort to keep them up to date. The spread sheets that are used end up being very complex, because automated testing is a complex thing to do, and as you add more functionality they get worse. They are useless, soulless and worst of all, take up all the time that manual testers have so that they stop doing the thing that manual testers do best, <strong>manual testing</strong>!

In our eagerness to move testing forward we are actually forgetting what the point of automated testing was in the first place. Automated testing was designed to make boring and monotonous regression tests a thing of the past. If the machine can rerun a series of known tests by itself and check that nothing has broken in the current build that frees the manual testers up to do the thing they do best; manual exploratory testing which is where you find all the bugs.

Automated testing checks that the functionality you have written works as you expect it to. Manual testing starts to push the envelope and use the program in ways it wasn’t intended to be used. Manual testing has no barriers, it isn’t constrained, and it is what <strong>will</strong> find holes in your application.

Some of the best manual testers I have worked with knew nothing about programming; they could not write automated tests and quite frankly had no interest in it. If I was building a new test team and could pick either two of them, or a team of forty automated testers, I would pick the manual testers any day of the week. They will find bugs, they will exercise the system properly and they will be of much more benefit to the project. I would still want automated testers as well to write the automated test framework and write regression tests, the point is that they would be doing this to free up time for the manual testers to go in and break the system in new and inventive ways.

To finish off I would like to make a plea:

Please don’t try to turn manual testers into data entry clerks, and don’t try and force them to become programmers; all you are doing is destroying the testing profession.

***Manual testers rock and are the heart and soul of testing, make sure you appreciate them!***
