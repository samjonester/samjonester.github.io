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

I can imagine paleontologists drooling over our applications' changelogs. I'm sure they wish they had a record of the exact environment that caused a recently uncovered Protoceratops to grow that very interesting third eyeball. They don't know when a helper method was added to give this horn-faced herbavore the ability to see oncoming predators, but unfortunately, made it trip over its feet running away.

We have this power. We can see when the responsibilities of a class grew. We can see how a method changed and a private method was created to hide complexity. We can see how a test was written, mocking that newly created method. We may not have been there when it happenend, but seeing that history can help us undestand a change.

## History is still being written

Understanding how motiviations and the current environment influence behavior is fascinating to me. It leads to some of my favorite conversations about code design. I love spreading the understanding that history is still being written. Unlike that ancient Protoceratops, we can reshape our flaws.

We can create an environment that motivates change.

## Questioning responsibility

One conversation I continue to enjoy is about **responsibility**. The conversation really boils down to one question.

> Should it really be the responsiblity of X to do Y?

From that general structure, there are many ways to ask questions about responsibility.

**Should an account really answer a question about a user?**
``` ruby
class Account
  def user_valid_admin?
    !@user.deactivated_at && @user.admin
  end
end
```

**Should an Order really know so much about a Camera?**
``` ruby
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
``` ruby
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
Fortunately for us, questions about responsibility still work very well on code that's already been written.

## Understanding change

All it takes is a reflective question and we can begin to understand. We can define the purpose of a class. We can decide how to limit complexity.

A question can help identify the flaws that we know exist and start to solidify the changes that we already know we want to see.

> We can understand the changes we would like to see in our code.

## Redefining responsibility

Our poor dinosaur friend didn't have a chance to move it's misplaced eyeball. This is software, though. We don't have the normal constraints of the natural world. We're only constrained by our ability to envision where responsibility should lie.

Where should it lie, then?

**We can ask an object about itself**
``` ruby
@user.valid_admin?
```

**We can tell an object to perform an action for itself**
``` ruby
@camera.package_size
```

**We can create classes and methods whose name's reflect their purpose**
``` ruby
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

-------------------

Pick a flaw and ask some questions.
