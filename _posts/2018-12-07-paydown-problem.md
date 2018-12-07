---
layout: post
title: "The Problem with Paydown"
description: "A tech-debt paydown plan is getting in the way of your team's happiness"
date: 2018-12-07
tags:
- leadership
- code-design
comments: true
share: true
---
> Your credit card debt is _out. of. control_. What do you do? Budget paydown!

In software development, we try to apply that same idea of budgeting paydown to technical debt in our applications. If the debt is too high, we feel the pain and try to eliminate it. We may declare bankruptcy and undertake a 😱 rewrite, or we may establish a budget and spend 10% or even 20% of our time on paydown. It's a concept that is easy to understand, especially when we liken it to something like credit card debt.

If you're thinking about starting, or have already instilled a paydown plan like this, then you're doing something awesome. You're clearly telling your team that you care about their happiness, and that's a good thing. Your paydown plan is coming from a good place! Here's the kicker, though. That plan is getting in the way of accomplishing your goal!

Paying down technical debt can't be compared so easily to credit card debt. Both can create a sometimes inescapable feeling of pressure, and both can have a huge impact on a person's happiness. But they aren't the same.

### Technical debt is hard to quantify

> How much extra work is this making, and how will not doing it impact us?

On a credit card statement, you see each purchase. You see the frivolous things you've bought and regretted. You see the necessary purchases that you couldn't avoid. Every purchase is listed as an absolute value which can be compared against your income to understand its impact on your lives.

Technical debt, however, is hard to quantify. You are unable to objectively measure how it weighs against your productivity. It's also very difficult to accurately predict the budget needed to pay down debt in an area of the code base when it's feeling overwhelming.

### Technical debt is difficult to repay incrementally

How do you contribute to the budget? Do you pay down on Fridays? Or Tueday and Thursday afternoons? Or even just squeeze paydown tickets into the board here and there? This means that you can't always focus on the activity as a single unit. Some things could easily span multiple weeks or months to complete.

How do you deal with the ongoing activity living in a branch separate from the evolving code? Every time you pick it back up you'll need to deal with a rebase to get it current, and then you'll be frustrated when the code you're trying to eliminate has spread into even more corners in the application.

Or maybe because of this pain, you only give yourselves permission to paydown things that are quick and easy. Larger, problematic architectural designs and data models may never be righted, regardless of the problems they are causing us day to day.

### Technical debt is about the present

How do you decide _what_ debt should be paid down? Is your team recording debt as they create it? Are they making a list as they run into it? Are you taking a poll of the parts of the code base that your developers wish they had permission to "fix"? This type of paydown is an attempt to make us feel better about things we aren't happy that we did.

Technical debt is about the present. It's something that makes the activities you'd like to do _now_ more difficult. Whether that activity is changing code, tracing a problem through the system, or just focusing on valuable work instead of tedious maintenance activities. Recording the debt you are introducing seems to solve an accounting problem, but it also makes the effort that would have benefited from the recorded activity something in the past, not the present.

When debt becomes a problem of the past, time passes before its paydown, and we will forget the pain. That paydown ticket will feel less important, and a lack of motivation will make it easier to send to the icebox when deadlines loom on "real work".

## The Problem with Paydown

Budgeting paydown may finally give you a chance to change those few lines of code that everyone hates, but those lines of code probably aren't the reason your team is feeling unhappy with the level of debt surrounding them. There's an assumption that creating a budget will erase debt, but it's not that straightforward. Establishing a paydown plan may actually create more debt and encourage a toxic atmosphere on the team to remain present.


### Paydown is a source of entropy

> Product: When can we release this?  
> Developer: Soon. It's done. I'm just cleaning up the code / writing some tests / reviewing with so-and-so  
> Product: Can we just release it and make another ticket to pay that down next week? We can add the extra work to our tech debt epic.

The idea that you can "finish" your activities after delivery gives a team license to redefine the concept of done. You begin to claim that things are done with the assumption that you can _really_ finish it later. Then if later never comes, all you've done is create entropy in your code bases. You're delaying the completion of this activity. You're sacrificing something. Maybe it's your confidence that this thing was built correctly, that the existing system hasn't changed unexpectedly. Maybe it's your ability to safely change this code in the future, when the original authors either aren't around or don't remember why untested, undocumented complexity exists.

You are hoping that you have the discipline to finish this task when there's more pressure to abandon the activity. Some paydown _will_ be deprioritized, regardless of your level of discipline, and what's the fallout if this activity is one of those things?

### Paydown puts dev & product at odds

> "I'll fight the business overlords for this 10%!"

Why should there be a fight at all? Your priorities should be a negotiation with the product owner based on mutual respect and trust. When you "fight" for paydown you're making an interesting statement to the business.

> When we said that thing you wanted was done, we weren't being truthful. It looks done to you, but it's not. Now, instead of working on the thing we committed to doing next, we're going back to cover our tracks.

Your paydown efforts are eroding the trust that allows us to have a productive negotiation with product. Instead of creating a shared understanding of the value you are creating, you're developing animosity for each other. You're declaring that the "other side" is short-sighted, and you need to take defensive action to protect yourselves.

Business will think that given the opportunity, you will waste time and money. That they need to keep tabs on your technological decisions so that you can be effective.
Your team will think that business decisions are forcing us to do things that are at odds with the longevity of your shared creation. You need to secretly "fix" their bad choices.

## Trust should be the priority

You and your team may have success with a paydown plan. However, if you don't feel that hapiness and trust are improving as a result, then maybe you're seeing some of the problems described in this post. If your paydown plan seems to be eroding trust between tech and product, then how can you help your team be happier without sacrificing valuable contributions to the business?

Become a conversation facilitator. Tell both product and tech "We trust you to make the best decision for your business". Instead of encouraging each side to protect itself from the other, encourage a healthy, honest negotiation between the two sides.

Allow tech to identify things that need to change to support the addition of new functionality, making the "paydown" a pre-emptive activity that can be scoped alongside the addition. Encourage business to convey the value added by functionality to help negotiate the size of the effort. Bring everything to the table as "the current state of affairs" and be honest about activities that would otherwise be happening in the dark.

Sure there will be times when the business pressures don't allow for the "ideal" technical solution, and there will also be times when the invisible modifications to the system seem to pause feature development. The goal is to create negotiations that are healthy so that everyone feels aligned with the business, even when it's not a win-win.
