---
title: "Factorix - A Factory Simulator"
date: 2021-10-19T09:22:28+02:00
draft: true
---

This series will chronicle my development process as I work on an academic project.
The goal is a simple configurable simulator for factory processes.

The scenario is simple enough: A factory creates a range products by performing a series of jobs on a number of machines in a shop.
That's it.
Optimizing the sequence of jobs on each machine so that the *makespan* - the time taken to complete each (or all) products ordered - becomes minimal is a [frequently](https://en.wikipedia.org/wiki/Open-shop_scheduling) [studied](https://en.wikipedia.org/wiki/Job-shop_scheduling) [problem](https://en.wikipedia.org/wiki/Flow-shop_scheduling) in computer science.

## What is Factorix?

**Factorix** simulates a "shop" given a configuration of machines, products, and job sequences.

After starting a simulation, a user can generate new product orders by sending requests via a REST API.
The user can then modify the task sequences for each machine to try and minimize the makespan manually.
If the user is so inclined, they might even create an algorithm to try and solve the optimization problem for them.
For those hopeless romantics, I plan to integrate some mechanisms into Factorix for outputting performance metrics like minimum, maximum, and average makespans, machine utilizations, etc.

## What's underneath?

I'll be going over a few of my design decisions for this project in the next section.
Aside from creating the domain model, my first question was: what programming language will I use?
I decided to go with one I have a couple of years of experience with, which is Java.
Now, domain modeling is obviously independent of the programming language.
Still, I prefer coding and writing tests as I start working on the model, so the decision to go with Java was a pretty quick and easy one.

![Coffee Beans](/coffeebeans.jpg "Powering this project in more ways than one.")

### UI

I'm reasonably confident that I want a simple web-based UI, but I know for certain that I'll be putting that question off for a while to come.
The application won't be that UI-intensive anyway, aside from some simple drag-and-drop interfaces and information boxes.

### Web Services

Design decisions regarding peripheral services like a REST API (and possibly OPC UA as an optional feature) are still pending.
I've used a cute little "microframework" called [Java Spark](https://sparkjava.com/) for a small REST service in the past, so that seems like a good enough option for this project.
[Eclipse Milo](https://github.com/eclipse/milo) is my go-to for implementing an OPC UA interface, but that feature is still up in the air.

![Spark Java Logo](/sparkjava.png "The Spark Framework. No, the other one.")

### Data Infrastructure

I do not see an immediate need for a database in this project, although a possible use might crop up at some point in the future.
Persisting a user's orders beyond a simulation session might be such a case, but a full database structure seems like absolute overkill on that front.
Even replaying order sequences is easier to implement with a file export/import.

A far more interesting infrastructure topic is how to get the "product-task-machine" configurations into the simulation.
A similar previous work project of mine used Excel to store the configurations, a design decision I have come to strongly resent.
It's not Excel itself that is bad and evil - it's Excel's Java API.
I'm not sure if there *is* a good way to process Excel workbooks in Java, but that API is definitely not it.
The search for an alternative is underway, but that will have to wait for another blogpost.

### Testing

Well, [JUnit](https://junit.org/junit5/). Obviously.
API testing may come into play at some point as well, and I will finally have a reason to break out my [Postman](https://postman.org/) skills.
The tests will co-evolve with my domain model, but I will try to have them done before moving on to my application layer features.
At some point in this project I might do a write-up on test-driven development.
If I manage to pull it off, that is.

## Coming Soon

Stay tuned for updates on this project.
I'll try to get them out on a weekly basis, or when there's major new features.

Until then!

~ nyando
