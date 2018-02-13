---
layout: post
title: "Questioning Responsibility"
description: "We're only constrained by our ability to envision where responsibility should lie."
date: 2018-02-07
tags:
 - code-design
comments: false
share: true
---

I can imagine paleontologists drooling over our applications' changelogs. I'm sure they wish they had a record of the exact environment that caused a recently uncovered Protoceratops to grow that very interesting third eyeball. They don't know when a helper method was added to give this horn-faced herbivore the ability to see oncoming predators, but unfortunately, made it trip over its feet running away.

We have this power. We can see when the responsibilities of a class grew. We can see how a method changed and a private method was created to hide complexity. We can see how a test was written, mocking that newly created method. We may not have been there when it happened, but seeing that history can help us understand change.

## History is still being written

Understanding how motivations and the current environment influence behavior is fascinating to me. It leads to some of my favorite conversations about code design. I love spreading the understanding that history is still being written. Unlike that ancient Protoceratops, we can reshape our flaws.

We can create an environment that motivates change.

## Questioning responsibility

One conversation I continue to enjoy is about **responsibility**. The conversation really boils down to one question.

> Should it really be the responsibility of X to do Y?

From that general structure, there are many ways to ask questions about responsibility. Fortunately for us, these types of questions work very well on code that's already been written.

**Should an account really answer a question about a user?**
```
class Account
  def user_valid_admin?
    !@user.deactivated_at && @user.admin
  end
end
```

**Should an Order really know so much about a Camera?**
```
class Order
  def package_size
    if @camera.compact?
      SMALL_PACKAGE
    else if (@camera.packaged_lenses.include? { |lens| lens.length_mm > 50 }) || (@camera.packaged_lenses.length > 2)
      LARGE_PACKAGE
    else
      MEDIUM_PACKAGE
    end
  end
end
```

**Should an Event really build the right Ticket?**
```
class Event
  def self.create_ticket(event_id, customer, seat=nil)
    event = self.find event_id
    if customer.vip?
      Ticket.create(event: event, type: BACKSTAGE_PASS)
    else if seat
      Ticket.create(event: event, type: ASSIGNED, seat: seat)
    else
      Ticket.create(event: event, type: GENERAL)
    end
  end
end
```
<br/>
The first time I a question like, I was blown away by how it affected my thought process, and I was blown away by the fact that *all I needed to do was ask*.

## Understanding change

As our code evolves, so do the responsibilities of the classes that we originally wrote. Even the cleanest methods with perfect names eventually change. Unfortunately, change usually means that the purpose of a unit has strayed from its original definition. I see this happen when code is added to a class simply because it already exists, because it's a convenient place to put another conditional.

All it takes is a reflective question and we can begin to understand, though. A gut check about responsibility often grows into a more thoughtful exercise where we can define the responsibility a class should have.


> Each class and each method should have a single purpose, and a name that focuses its responsibilities.

Questions help us pause to evaluate what's there and how it happened. We can understand the evolution of our code by looking at the changes in functionality. A change here, and another one there, and now the responsibilities of the class no longer match its name. We can identify the changes we already know our code needs.

## Redefining responsibility

Our poor dinosaur friend didn't have a chance to move it's misplaced eyeball, but this is software. We don't have the normal constraints of the natural world. We're only constrained by our ability to envision where responsibility should lie.

Where should it lie, then?

**We can ask an object about itself**
```
@user.valid_admin?
```

**We can tell an object to perform an action for itself**
```
@camera.package_size
```

**We can create classes and methods whose name's reflect their purpose**
```
class TicketBuilder
  def create(event, customer, seat)
    if customer.vip?
      Ticket.create_backstage_pass(event)
    else if seat
      Ticket.create_assigned_seat(event, seat)
    else
      Ticket.create_general_admission(event)
    end
  end
end
```

Each of these changes helps to redefine the responsibilities of a class or method. They help the unit remain focused on one thing.

**To create change like this, I use guidelines:**

- Each class and method should have one purpose.
- The responsibilities of the unit should be limited to achieve that purpose.
- The name of the unit should be accurate and provide focus.
- The unit should interact with other well named, focused components to clarify its purpose.

Guidelines like this help me answer my own questions and promote the right change. They helped me create the changes to the examples above. They help me create change with the teams I work with. They drive my favorite conversations about responsibility.

### The Paleontologist Developer

The mark of a paleontologist is their habit of questioning the past.

As our code ages it will gain complexity and stray from its original purpose. A thoughtful developer will question the evolution of code in the same way as a paleontologist. Both want to understand how change happened, but only one has the opportunity to *make* change happen. Take advantage of the unique malleability of the products we produce, and ask questions.

All we need to do is ask.
