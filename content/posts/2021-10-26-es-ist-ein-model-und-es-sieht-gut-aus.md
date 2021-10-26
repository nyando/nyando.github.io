---
title: "Es ist ein (Domain) Model und es sieht gut aus"
date: 2021-10-26T09:22:28+02:00
draft: false
---

If you speak German, you probably understood the slightly modified Kraftwerk lyric in the title.
If you don't speak German, you'll probably have noticed the words "domain model".
And if you're still reading, it means you ignored the extraneous stuff you didn't understand around those two words and simply clicked because "domain model" was interesting to you.
Congratulations, you're already in the correct mindset for this post.

## What is a Domain Model?

A domain model is somewhat redundantly defined as a model of a software's problem domain.
Since that's not helpful at all, let's break it down.
Software is created to solve problems.
These problems possess components and contexts that factor into their solutions.

Now, that's still pretty abstract. So let's make it concrete and use my burgeoning factory simulator as an example.
I implicitly gave Factorix's problem statement in the last post.
Here it is, slightly reformulated:

> Given a configuration of workplaces, products, and processes, simulate a production environment with an input of orders and task sequences from an external user.

This statement is very dense, but the idea of what a problem is should have become clearer.
More importantly, it's stated in a way that gives us an idea of the elements that make up our **problem domain**.
Specifically words like *workplace*, *product*, *order*, *task*, etc.
These are the **entities** that we use in our domain model to recreate the processes happening in a real factory.
Object-oriented languages like Java allow us to create abstract models of these entities and have them interact according to our **business logic**.
Ideally, our business logic is similar to the interactions that happen in real life (if maybe more abstract).

## So what do we do with this?

![The Factorix Domain Model](/imgs/factorix_domain.png "The Basic Idea")

You'll notice a few differences from the problem statement.
There are no processes, and there's something called a "buffer".
A *process* can be viewed as a relationship between a workplace and a product.
A workplace performing a process on one type of product may take a different time performing the same process on a different type.
On the other hand, a *buffer* is actually a component of a workplace, it's just not included in the (admittedly shortened) problem statement.
Input buffers store incoming products and release them into the workplace when it's time to work on them.
Output buffers store outgoing products waiting for a ride to the next workplace.

Speaking of which: *Transportation* is not in this diagram at all.
There's a reason it's not in the problem statement.
We will obviously have to get our products from workplace A to workplace B, but we'll accomplish that with a service.
And since transportation isn't the focus of this application, we won't make it more complicated than it needs to be.

## This Class Does Not Spark Joy

As I alluded to in the beginning of this chapter, ignoring extraneous stuff is very useful when it comes to domain modeling.
You want to focus your model on the things you need to solve your problem.
In this case, since we want to simulate a production environment, let's go over a list of what a production environment needs:

* `Workplace` - Seems like a no-brainer. Where else would we work?
* `Buffer` - Buffers are components of a workplace. It's debatable whether or not they need a class of their own, but I'd like to separate concerns of work and transport, and a buffer allows me to do just that.
* `Order` - Orders serve as organizational units. We want to know when an order is completed, so we'll have to organize our products and tasks correspondingly.
* `Product` - Again, duh. Or, more philosophically: What would a workplace be without a product?
* `Task` - This one is a little debatable. In this model, a task is an expression of a process that needs to be done on a series of products to complete an order.
Workplaces organize their tasks in a queue and work through them sequentially.

Now obviously someone could come up to us and ask something like, "hey, where are the storage containers for the finished products?"
The answer is that they're not part of our problem domain. So we don't care about them.
The same second an order's last product falls out of the output buffer of a workplace, that's when we stop caring about it (well, aside from noting its makespan).

## So what's next?

So say we have all these classes and their attributes modeled out, we have their interactions mapped onto the class methods, etc.
We still need some way to connect them.
We need a way to create a batch of products when a new order comes in, we need to distribute tasks to their respective workplaces, and so on.
Classes that make these entities work together and interact with one another are called *services*.
In next week's post (fingers crossed!), we'll be discussing the details of what needs to happen when a new order comes in.

Until then, have a nice week!

~ nyando
