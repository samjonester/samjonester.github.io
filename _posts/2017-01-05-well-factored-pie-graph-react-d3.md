---
layout: post
title: "A Well Factored Pie Graph: React + D3"
description: "A technologist's most powerful skill is the ability to analyze and leverage a tool's strenghs. React and D3 have different strenghs, which work well together to build a complex visualization.<Paste>"
date: 2017-01-05
tags:
 - React
 - D3
 - JavaScript
 - code-design
comments: false
share: true
---
## What and Why

Our goal is to analyze two popular tools to find their strengths, then build a well factored integration between the two tools. As an example of this technique, we will create a well factored pie graph. We will focus on separating the responsibilities of state management and graph presentation by using tools that shine for each purpose separately. We will use [React][React] to manage state and [D3][D3] for presentation.

The live example below was built in an [example repository][repository]. I'm going to focus on overall strategy and critical analysis, rather than provide a comprehensive code listing. Feel free to follow along with the code example.

<figure>
  <div id="graphic"></div>
  <script src="/assets/react-d3-example.js"></script>
  <figcaption style="text-align: left;">
    A pie graph of pies. How original.
  </figcaption>
</figure>

## Incoming Data

``` javascript
const data = [{
    name: 'Apple',
    percent: 30,
    color: '#E7BD2C'
  }, {
    name: 'Cherry',
    percent: 60,
    color: '#E73E2'
  }, {
    name: 'Pumpkin',
    percent: 10,
    color: '#E7932C'
  }]
```

We will provide data to fill our pie graph. Each piece of data will contain it's name, color for the graph, and the percent of the graph it will represent.

## Start With State

** We will make our graph interactive by highlighting which slice we've clicked. **

When considering this requirement, we will need to store state within our graph. Our state will be the data to build the graph, and also, a record of which slice has been selected. The ability to manage state in this way is the main strength of [React][React]. React provides 3 important features for this graph.

1. We can store the incoming data as state within the component to eventually provide to the presentation piece.
1. We can provide a function to update the selected slice and provide it as a click event to the presentation piece.
1. We can test the state change in isolation, and verify the interaction with our presentation piece.

The last feature, test isolation, is important to our goal of creating a well factored interaction between components. If we separate the presentation piece and treat it as a black box, we can have a more focused test for our component. Our component will be smaller and easier to understand. It will have the single responsibility of managing the collaboration between our state and our presentation piece.

## A Collaborating Component

Let's begin examining how our React Component is managing the collaboration between our data and our presentation piece. We'll take a look at how we store the incoming data, after first preparing it for our presentation piece.

``` javascript
import { calculateAngles } from './pie.d3'

export default class Pie extends React.Component {
  constructor(props) {
    // ...
    this.state = {
      data: calculateAngles(this.props.data).sort(d => d.selected)
    }
  }
  // ...
}
```

### Separate the Calculation

We must transform the percentages into angles between 0 and 2 PI to satisfy the presentation piece. However, you don't see a hint of that algorithm here. The calculation could happen in-line, but let's consider how that code should be factored. 

Inlining the calculation would distract from the responsibility of the component. When we separate it from the component, then the component does not need to know the algorithm to prepare the data. It's only responsibility remains to coordinate the incoming data and our presentation piece. Most importantly, the calculation can be tested in isolation. Our component only needs to verify the interaction with the calculation.

## Presentation as a Black Box

When our component is mounted, and also when the state has changed, we are sending our current state as data to the presentation piece. At this point our component doesn't even know that we're using D3 to render the pie graph, just that it has a collaborator to do the hard work.

``` javascript
import { loadGraphic } from './pie.d3'

export default class Pie extends React.Component {
  // ...
  createGraphic() {
    loadGraphic({
      rootNode: this.refs.arc,
      data: this.state.data,
      onSliceClick: this.selectData.bind(this),
      height: this.props.height,
      width: this.props.width
    })
  }
  // ...
}
```

If we look at the function call to `loadGraphic`, which renders the pie graph, we've got a clean ["black box"][black-box] interaction.

* We're providing a state manipulation function as a click event.
* We're providing a rendered DOM node as the root of the pie graph.
* We're providing the current state of the component as the data for the pie graph.

This allows us to perform interaction based testing with the presentation piece. We can focus on one responsibility still, coordinating our state with our presentation piece.

## Rendering A Pie Graph

React has the ability to build an [SVG][SVG], since `<svg>` can easily be represented with [JSX][JSX]. However, [D3][D3] has a large set of mature libraries for building complex visualizations. For example, we will be using the [d3-shape][d3-shape] library to calculate the paths of a our slices within our pie graph. Defining the paths is a complex calculation, which D3 provides an easy to use API for.

React and D3 both have impressive strengths that apply directly to the problem being solved, but in different ways. It is best to find a way to integrate the two tools, rather than coerce one tool into doing a job that it is not ideally suited for. This integration could be complex, tightly coupled, and difficult to change in the future. However, by factoring them properly, we are creating a partnership that allows both tools to focus on what they are best at.

### A Clean Entry Point

When designing a loosely coupled component, as we are with our presentation piece, it's important to define an easy to interact with entry point. You saw in the `createGraphic` function of our component that the public API was clean and easy to call.

Just as important, though, is having a small, simple, and easy to understand function body within that entry point. Let's take a look at our `loadGraphic` function for an example.

``` javascript
import compose from 'lodash/fp/compose'

export function loadGraphic({rootNode, data, onSliceClick, height=600, width=600}) {
  compose(
    createLegend(data, height, width),
    createChart(data, height, width, onSliceClick),
    createShadow,
    setContext(height, width)
  )(rootNode)
}
```

We're using a functional programming concept called `compose` to define what it means to load our graphic. Compose gives us the ability to glue together a series of steps into a larger piece of functionality. The important take-away here is that each step in building the graphic says WHAT is happening, and not HOW. It's composed into a larger piece of functionality on `rootNode`.

### The Power of Pie

To properly understand the value in choosing to use D3 for presentation, we will need to take a closer look at the API it provides for defining a pie graph.

``` javascript
import { arc as d3Arc, pie as d3Pie } from 'd3-shape'

function createChart(data, height, width, onSliceClick) {
  return function(rootNode) {
    // 1. Transforming the data within the chart
    const chart = rootNode.append('g')
      .selectAll('g')
      .data(pie(data), d => d.data.name)

    // 2. Adding our slices to the chart
    chart.enter().insert('path')
      .attr('d', slice(radius(height, width)))
      // ...

    chart.exit().remove()

    return rootNode
  }
}

function radius(height, width) {
  return width / 2
}

function pie(data) {
  return d3Pie().value(d => d.name)(data)
}

function arc(radius) {
  return d3Arc().outerRadius(radius * 0.4).innerRadius(radius * 0.2)
}

function slice(radius) {
  return arc(radius).startAngle(d => d.data.startAngle).endAngle(d => d.data.endAngle)
}
```

There two important things happening in the `createChart` function that are the crux of the value provided by D3.

**1. Transforming the data within the chart**

When we define the chart, we will create a container group (`<g>`) to hold our chart. We will assign the data for this group by using a function `pie`. This provides a declarative API to define the data being iterated over, and abstract away any nitty-gritty details about its structure. Noticing a pattern?

**2. Adding our slices to the chart**

For each piece of data defined by the `pie` transformation, we will define a [path][path]. This path needs daunting calculations to be performed. Luckily D3 has taken care of this task for us using the `slice` function! If you don't believe this is valuable, take a look at the HTML source for the example above and try to understand the path being defined.


# In Summary

Well factored code allows us to respond to strengths and weaknesses in our tools. We can integrate our favorites together more easily, and with better success when we understand how to do a few simple things!

* Define Separate Responsibilities
* Retain Focus Inside Single Units
* Define Clear Boundaries
* Create Clean Public APIs

There is a [repository][repository] available with a full code example for this post. It is under the MIT license, so please use it to learn, share, or expand upon for your own React+D3 pie graph. If you've got any questions, or ideas on how to improve the code, please open an issue!


[React]: https://facebook.github.io/react/
[JSX]: https://facebook.github.io/react/docs/jsx-in-depth.html
[SVG]: https://developer.mozilla.org/en-US/docs/Web/SVG
[D3]: https://d3js.org/
[black-box]: https://en.wikipedia.org/wiki/Black_box
[d3-shape]: https://github.com/d3/d3-shape
[path]: https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths
[repository]: https://github.com/samjonester/react-d3-blog-example
