---
layout: post
title: "Accessibility Techniques"
description: "Impactful Tools and Techniques to Get You Started"
date: 2017-04-28
tags:
 - accessibility
comments: false
share: true
---

Accessibility techniques are a way to create a more [usable experience][accessibility-usability] for all users. An accessible experience can be viewed as a [continuum][accessibility-continuum], and this post is an introduction to techniques that span that continuum. We will start with accessibility techniques that are most likely to fix issues that block users with disabilities from successfully navigating your application. As we get comfortable with the most impactful techniques, we will move into techniques that create a more inclusive experience and provide equal access to even more people.

When I started learning about accessibility techniques, I was quickly overwhelmed and found it difficult to find a good jumping-off-point that humanized the techniques. These techniques may be "just the beginning", but they will provide the most impact during initial development, and are where I like to start.

## Keyboard Navigation

Testing and improving [keyboard accessibility][keyboard-accessibility] can unblock navigation for users with motor disabilities. It can also be a good gut check when creating content to be navigable by users relying on screen readers. Ensuring your page is navigable by keyboard is the most impactful action to take. Start here and build more advanced techniques on top of this base.

I like to think about keyboard accessibility as two categories. The first, **focus**, enables keyboard navigation on the current state of the page. The second, **toggling visibility**, ensures that when content is dynamically added or removed, keyboard navigation is still meaningful.

### Focus
The `TAB` key is used for navigation through actions within a web application. As you hit tab, the next element in the tab order receives focus and can accept action.

- **Favor Native Elements**<br/>Some elements naturally receive focus when `TAB` is used to navigate to them. The most widely used examples are `<a>`, `<button>`, all types of `<input>`, and `<textarea>`. Focus works splendidly when these elements are [used appropriately][native-elements]. However, when a `<div>` looks like a button, it does not receive focus, and blocks access to content presented by clicking the "button".
- **Provide a Tabindex When Necessary**<br/>When a native element isn't an option, a `tabindex="0"` will [enable an element to receive focus][tabindex] in the normal tab order. A `tabindex="-1"` will remove an element from the navigation flow.
- **Supplement Hover Actions with Focus**<br/>Controls should not be hidden behind hover actions, like those to display navigation menus. Hover actions cannot be triggered while navigating the page with a keyboard, so supplement hover actions with focus actions to ensure that your content is able to be presented to more people.
- **Design Useful Focus Indicators**<br/>The focused element should [stand out][accessible-focus] while navigating with the keyboard. A good trick is to close your eyes, hit tab a couple times, then determine if the focused element is obvious to pinpoint.
- **The Tab Order Should Be Natural**<br/>As you navigate the DOM using the keyboard, the order of focused elements should be natural and [follow the presentation][focus-order] of content. No one wants to jump from the logo, to the footer, and then back up to the navigation bar as they tab through your application.

### Toggling Visibility
As dynamic content is displayed or hidden, ensure that keyboard navigation is still meaningful.

- **Preserve a Meaningful Tab Order**<br/>Newly presented content should gain the focus of both the user and the keyboard. It should be added to the DOM in the most logical order for [presentation to screen readers][dynamic-order]. Move focus outside the hidden content after closing, because it's very disorienting when focus is completely lost from the page.
- **Close with Escape**<br/>When new content is presented on screen with modals, slide out side-bars, etc, ensure there is an "escape path" to [close the content][dismissing-content] by pressing the `ESC` key. 
- **Buttons vs Links**<br/>What would happen if a screen reader user asks for support and is told to click a button in the UI, but it's actually a styled link? Buttons and links have [specific purposes][buttons-links]. The decision to use one over the other does have an impact on the conveyed meaning of an action.

<br/>
## Extras
<hr style="border-width:3px"/>

*In my mind this line represents the division between ensuring basic navigation and providing an inclusive experience. After becoming comfortable with the most impactful techniques, these are great places to continue learning.*
<br/>

### Getting Started with Aria

[Aria][aria] is a set of html attributes that define Accessible Rich Internet Applications. There are many aria attributes, and each provide a way for screen readers to convey semantic meaning to users with vision disabilities. I started with these attributes while learning about how to add contextual information using Aria.

- **Aria landmarks**<br/> When visually displayed content is presented through a screen reader, the main semantic structure of the document is lost. Defining [landmarks][aria-landmarks] within the page will convey this meaning to a screen reader, so that areas of the page like navigation and content have meaning to users with visual disabilities as well.
- **Aria roles**<br/> [Aria roles][aria-roles] provide more context about how an element is used, like a `menu` with `menuitems`. This is especially important when elements aren't used in their intended way. This provides a way for a screen reader to understand that a `div` that looks like a button behaves like one, too.
- **Aria states**<br/> [Aria states][aria-states] provide more context about the current characteristics of a dom element to a screen reader to convey that a `div` that looks like a checkbox is `checked`.

### Visual Representation

- **Favor higher contrast**<br/> Text and information that is conveyed visually should have high [color contrast][contrast]. This will ensure that users with vision disabilities, users with poor eyesight, or users on a mobile phone in the sun can understand the information. Using patterns in addition to colors will also ensure that users with colorblindness are able to understand information presented with graphics.
- **Provide alternative text**<br/> [Alternative text][alt-text] takes the form of `alt` attributes on images, transcripts for videos, tablular data in addition to a bar chart, or even just explaining a graphic in a surrounding paragraph. The purpose is to provide the context and information to screen readers. As an added benefit, alternative text increases the searchability of the content being presented to a user.

## Tools I Recommend

Static testing is nice, but doesn’t replace manual testing. Static tests can only look at the current state of the DOM, and are missing context to make determinations about usability. There are some checks that *can* be very effectively analyzed by a static tool, though. Here are the tools that I recommend adding to your accessibility test suite for static testing.

- The [aXe][axe] tool suite created by [Deque][deque] and centered around the [aXe core][axe-core] JavaScript library.
  - [aXe Core JavaScript Library][axe-core]
  - [aXe Matchers Ruby Library][axe-matchers]
  - [aXe Chrome Extension][axe-chrome]
- The [Pa11y][pa11y] Command Line and JavaScript accessibility testing pal.
- The [tota11y][tota11y] bookmarklet created by the [Kahn Academy][kahn].

Start taking action today, as any action will improve the [usability][accessibility-usability] of your site.

[accessibility-usability]: /posts/2017-04-07-accessibility-is-usability.html
[accessibility-continuum]: /posts/2017-04-07-accessibility-is-usability.html#continuum
[keyboard-accessibility]: https://www.nngroup.com/articles/keyboard-accessibility/
[native-elements]: https://marcysutton.com/links-vs-buttons-in-modern-web-applications/
[tabindex]: http://accessibleculture.org/articles/2010/05/tabindex/
[accessible-focus]: https://www.deque.com/blog/give-site-focus-tips-designing-usable-focus-indicators/
[focus-order]: https://www.w3.org/TR/UNDERSTANDING-WCAG20/navigation-mechanisms-focus-order.html
[dynamic-order]: https://www.w3.org/TR/WCAG20-TECHS/SCR26.html
[dismissing-content]: https://www.smashingmagazine.com/2014/09/making-modal-windows-better-for-everyone/#dismissing
[buttons-links]: https://marcysutton.com/links-vs-buttons-in-modern-web-applications/
[aria]: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA
[aria-roles]: http://www.webteacher.ws/2010/10/14/aria-roles-101/ 
[aria-states]: http://www.webteacher.ws/2010/10/15/aria-states-101/
[aria-landmarks]: https://dequeuniversity.com/assets/html/jquery-summit/html5/slides/landmarks.html
[contrast]: https://www.w3.org/TR/UNDERSTANDING-WCAG20/visual-audio-contrast-contrast.html
[alt-text]: http://webaim.org/techniques/alttext/
[deque]: https://www.deque.com/
[axe]: https://www.deque.com/products/axe-core/
[axe-core]: https://github.com/dequelabs/axe-core#axe-core
[axe-matchers]: https://github.com/dequelabs/axe-matchers#axe-matchers
[axe-chrome]: https://chrome.google.com/webstore/detail/axe/lhdoppojpmngadmnindnejefpokejbdd?hl=en-US
[pa11y]: http://pa11y.org/
[tota11y]: http://khan.github.io/tota11y/
[kahn]: https://www.khanacademy.org/
