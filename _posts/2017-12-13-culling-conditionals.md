---
layout: post
title: "Culling Conditionals"
description: "Abstractions to Keep the Population at Bay"
date: 2017-12-13
tags:
 - code-design
 - JavaScript
 - function-programming
comments: false
share: true
---

Growing up near the Great Lakes, I'm well aware of the threat that [Asian carp](https://www.scientificamerican.com/article/asian-carp-woes/) are posing on the polluted lakes we endearingly call the "North Coast". It seems that these carp can't be stopped. They're breeding like crazy in an environment that isn't equipped to keep their population in check. It's so bad that [underwater electric barriers](https://www.jsonline.com/story/news/local/wisconsin/2017/08/07/army-corps-engineers-plans-new-defenses-block-asian-carp-reaching-great-lakes/546376001/) have become viable solutions to the problem.

I can't help but draw parallels between these carp and conditionals in code I've seen over the years. They're always introduced for a good reason, whether they're controlling parasites or controlling application flow. Once they start to spread, though, they tend to take over.

## Conditionals Spread

Are conditionals really as evil as an invading species of bottom feeders? To answer that question we need to look at how software changes over time. That requires us to think about how __*people* change software__ over time.

People usually make the most obvious and convenient change. When there's delivery pressure, we do what's easiest. This is especially true when we aren't confident that our change is isolated. When we're worried that we may break other code, we do what's most convenient and seems least "dangerous".

Unfortunately, this means that conditionals tend to grow. What's the easiest change when the complexity of a function is already through the roof? We add one more branch to the conditional and cross our fingers.

## Conditionals are Confusing

Conditionals mix multiple levels of abstraction in one neat little package. Here's a list of things that I have to keep in my head while I'm reading a conditional:

1. The shape of the conditional to understand the available paths through the application.
1. The scope of multiple variables as they're used within the conditional.
1. The values that are used to determine which path to follow.  
   \* _This often means understanding the structure of a deeply nested object._
1. The actual code being executed within each path.
1. How each of these specific actions affect the larger system since none of them have names.

Just thinking about all the things I need to think about leaves me exhausted. This is another reason why conditionals spread. After getting a grasp on all that information, I just don't have any more room to plan a refactor!

## Abstractions to Keep the Population at Bay

Just like those carp, conditionals should be carefully culled. Unfortunately, Slack doesn't have integrations for [electric barriers or poison pills](http://www.chicagotribune.com/news/local/breaking/ct-lake-michigan-asian-carp-met-20160822-story.html). There are a few patterns that I regularly use instead of conditionals. Before blindly adopting any of these techniques, ask if the code can be written [without a conditional](http://michaelfeathers.typepad.com/michael_feathers_blog/2013/11/unconditional-programming.html).

These 2 techniques build on [All the Little Things](https://www.youtube.com/watch?v=8bZh5LMaSmE). They are ways to design components that have a singular focus, rather than a list of responsibilities like we see above.

Each technique has a specific problem it solves. Each proposed alternative has a few important things in common.

- The new abstraction has a single purpose. Choose and execute the appropriate path(s) through the application. That's it. Fake paths can be provided for testing the abstraction so details about a specific path aren't complicating the test.
- Each path has the same 2 methods. One to test whether it should be executed, and one to execute the action. This allows each piece to be defined and tested in isolation. Also we gain valuable names describing each path, helping other developers understand its purpose.

### Alternative to if / else if / else

**The problem:** We only want to execute one path through the application.

```javascript
export default function walkDog (dog) {
  if (dog.hasEatenRecently()) {
    dog.walk(length: LONG_WALK)
  } else if ((time.readyForBed() && dog.hasWalkedRecently()) || time.aboutToLeave()) {
    dog.walk(length: JUST_POP_OUTSIDE)
  } else {
    dog.walk(length: AROUND_THE_BLOCK)
  }
}
```

**The Solution:** Find the first matching path, then execute it.

```javascript
import * as pooWalk from './poo-walk'
import * as beforeBed from './before-bed'
import * as aboutToLeave from './about-to-leave'
import * as normalWalk from './normal-walk'

const walkingPatterns = [ pooWalk, beforeBed, aboutToLeave, normalWalk ]

export default function walkDog (dog, walkingPatterns = walkingPatterns) {
  walkingPatterns
    .find(pattern => pattern.matches(dog))
    .execute(dog)
}

// ------------- ./normal-walk.js ------------- //
export function matches (shoppingCart) { ... }
export function execute (shoppingCart) { ... }
```

### Alternative to multiple ifs

**The problem:** We only want to execute each appropriate path through the application.

```javascript
export default function applyDiscounts (shoppingCart) {
  const discountedTotal = shoppingCart.total;

  if (shoppingCart.promotions && shoppingCart.promotions.length >= 0) {
    discountedTotal -= shoppingCart.promotions.each(promo => promo.apply(shoppingCart))
  }

  if (shoppingCart.user.credit) {
    discountedTotal -= shoppingCart.user.credit
  }

  return discountedTotal
}
```

**The Solution:** Find all matching paths, then execute them.

```javascript
import * as cartPromotions from  './cart-promotions'
import * as userCredits from  './user-credits'

const discountCalculators = [ cartPromotions, userCredits ]

export default function applyDiscounts (shoppingCart, discountCalculators = discountCalculators) {
  return discountCalculators
    .filter(calculator => calculator.canBeApplied(shoppingCart))
    .reduce((discountedTotal, calculator) => discountedTotal - calculator.apply(shoppingCart), shoppingCart.total)
}

// ------------- ./cart-promotions.js ------------- //
export function canBeApplied (shoppingCart) { ... }
export function apply (shoppingCart) { ... }
```

