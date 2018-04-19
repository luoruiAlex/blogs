## display: flex | -webkit-flex | inline-flex
## 特点：clear、float、vertical-align属性会失效

## 组成
- container
  - main start => main end == main axis
  - cross start => cross end == cross axis
- item: main size X cross size

## container
- flex-flow: flex-direction + flex-wrap
- flex-direction: ***row*** | row-reverse | column | column-reverse
- flex-wrap: ***nowrap*** | wrap | wrap-reverse
- justify-content: ***flex-start*** | flex-end | center | space-around | space-between
- align-items: felx-start | flex-end | center | baseline | ***stretch***
- align-content: flex-start | flex-end | center | space-between | space-around | ***stretch***

## item
- order: <int: 0>越小越靠前
- flex: flex-grow + flew-shrink + flex-basis
- flex-grow: <number: 0>
- flex-shrink: <number: 1>
- flex-basis: <length | ***auto***>
- align-self: ***auto***(extend align-items) | flex-start | flex-end | center | baseline｜stretch
