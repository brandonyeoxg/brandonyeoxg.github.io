+++
draft = true
date = 2021-12-05T16:14:50+08:00
title = "GO Tidbits Antipattern Part 1"
description = "Sharing of anitpatterns in golang"
tags = [golang, tidbits, antipatterns]
categories = [golang]
series = [golang, tidbits, antipatterns]
+++
Many engineers that I've come across in my journey often mention that software engineering is a craft. When I read engineering books such as [Code Complete](https://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670) the concept of craftmenship and the gathering of "tools" for the "toolbox" was highlighted several times. That got me curious on what sort of "tools" that I could gather.

In software engineering there is the concept of design patterns. These patterns. broadly speaking, are commonly used solutions to tackle problem spaces that they were design for. Suffice to say any engineer worth their salt would have come across such patterns; **factory**, **strategy** or **command** just to name a few.

When a design pattern is implemented for problems it is not meant to solve, there is bound to be code smell. This makes the codebase harder to maintain and might even introduce unintended behaviours. When this arises, the pattern is now considered an antipattern.

Knowing how antipatterns look like should provide the knowledge to avoid them and thus make our code cleaner and simpler.
This antipattern is the "tool" that I would like to include in the engineering "toolbox" of mine. I will be starting a series dedicated to documenting antipatterns as a way for me to document the antipatterns that I've come across, learning along the way and I hope it will help you learn as well.

> The language that we will be covering is for GO, however I do believe that the knowledge should be tranferable to other languages as well.

Let's start off with the simplest antipattern to spot: boat anchor.

Boat anchor a is piece of code that is no longer relevant in the code but is still left if there in hopes that there might be "some use" for it later on. As years go by and engineers come and go, the belief that this has "some use" gets lost. Newer engineers are reluctant to change and choose to work around these boat acnhor, furthering bloating the code base.

One such example:
```go
if config.ExperimentA() {
  doOperationA()
}
```
This is all well and good, however experiment A was a success and s currently used in the business flow.

A new engineer who is helping to add feature might look at this and think that this is probably something that is important. They might ask their experienced colleagues (that is if they have the relevant context) but more likely than now, they would work around this in hopes of not causing any business outage.

They might probably do something like:

```go
if config.ExperimentA() {
  doOperationA()
}

if config.ExperimentA() && config.RolloutNewFeatureA() {
  doNewFeatureA()
}
```

Notice the code bloating on the above? It is clear that the code will become harder to maintain in the feature if this keeps out.

Contrasting this to the above:
```go

doOperationA()

if config.RolloutNewFeatureA() {
  doNewFeatureA()
}

```

Which do you think is more maintainable? The code looks much better by simply cleaning up after ourselves when we successfully finished rolling out a feature or an experiment.

As the saying goes less is more.
