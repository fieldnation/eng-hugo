+++
author = "Tim Mader-Brown"
comments = true
date = "2019-03-06"
draft = false
image = "images/bug.png"
menu = ""
share = true
slug = "Launching-MicroMN"
tags = ["Field Nation", "QA Automation Architect", "test automation"]
title = "Making Test Data"

+++

# Making Test Data


Hello World!

Today I bring the Field Nation blog back to life after a 2+ year coma. What's the topic for this sudden revival?  One of my very favorite topics….Creating Test Data.  

![tent in the tundra](./../../static/images/ice_tent.jpg)

What does a tent sitting in the middle of the tundra have to do with creating test data? This illustrates a very important point in regards to testing.  When someone wants to test this tent, they would set it up in different climates, at different times of the year, and maybe using some different add-ons for the tent, for example, northern Minnesota, January 1-5 using the winter shield (I'm not sure what a winter shield could be).  The tester would bring the supplies to that site, set up the tent and report the findings.  It’s not likely that the tester would be manufacturing the tent from scratch, the pre-condition for the testing is to already have the tent ready for use.

Our software test should be no different than this.  I’ve seen it time and time again where a test case will run through a handful of tasks, create a user, update that users profile, put the user into some sort of group, and after all that is done then the test will actually start.  This is especially painful with its manual tests doing this because humans have a tendency to be inconsistent with their approach, as well as the fact that its borderline torture to repeat the same tasks like this over and over and over and….you get my point.

We solved this problem at Field Nation using a project that we call the Test Data Generator.  Rather than having a manual test or an automated test click through all the pre-conditions needed to complete the test, we can make a single all to the test-data-generator that will set all of that data to the state we need for the test.  Here is another simple example, let’s say Facebook wants to test that a user can join a group.  If they have the test-data-generator (and I’m sure they have something equivalent) they would have the test began by running something that would generate a brand new user either by directly interacting with the database or via an API, then after the user is created then they could use that user in an integration test and go through the motions of adding the user to the group then checking that it was successfully added to the group.

It’s great because using this tool makes all of our tests consistent and begin at the correct baseline.  Also, since we made a web service for the test-data-generator we thought “Hey, maybe if we made a UI for this then other people, like support or even sales could use it”.  So we created a UI for it that looks like this…

![test data generator ui](./../../static/images/tdgUI.png)

This is all very new at Field Nation, so I'll report back more findings as they come up
