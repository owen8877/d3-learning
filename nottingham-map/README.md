# [Map of Nottingham](https://bl.ocks.org/owen8877/6de893c1f542bd8dcba74d76b1203043)
![preview.jpg](By xDroid)

_Data based on our mcm thesis._

## Introduction
The map is drawn for our mcm thesis (of course not for this time), to intuitively show the distribution of blocks with different functions. 
We also collect data on public transportation (mainly from bus station numbers).

The original code was written under node.js environment with the help of `d3-node`, rendering a `svg` file which can be easily converted to `png` or other picture types. 
Since I want to learn `d3.js` thoroughly, so I tranport the code to browser platform. 
(It is obvious that the work is very easy.) 

## Reading the code
We have a template to minify the trouble of writing htmls every time.
```html
<!DOCTYPE html>
<head>
<meta charset="utf-8">
<link rel="stylesheet" href="//danlevan.github.io/google-material-color/dist/palette.css" type="text/css" />
<style>
</style>
</head>
<body>
<script src="//d3js.org/d3.v3.min.js"></script>
<script src="//danlevan.github.io/google-material-color/dist/palette.js"></script>
<script></script>
</body>
```
Besides _d3.js_, I also use js and css files from [_google-material-color_](https://github.com/danlevan/google-material-color) repo to simplify color picking work.

### Where loads the data ?
Good question. Here.
```javascript
d3.tsv("nottingham.tsv")
.row(d => {
  d.x = +d.x
  d.y = +d.y
  d.type = +d.type
  d.development = +d.development
  d.bus = d.bus === "true"
  return d
})
.get((error, data) => {
//...
}
```

### The processing work
The work is mainly divided into two parts: displaying the blocks and drawing the legends.
#### Displaying blocks
These two functions are related: `blockBackgroundLayer(blockSelection)`, `busLayer(blockSelection, data)`. And they do the following things.
```javascript
function blockBackgroundLayer(blockSelection) {
  blockSelection
    .append('rect')
      .attr('x', blockDimension.x)
      .attr('y', blockDimension.y)
      .attr('width', blockDimension.width)
      .attr('height', blockDimension.height)
      .style('fill', d => {
        return palette.get(typeMapping[d.type].color, developmentMapping[d.development])
      })
}
```
(If you ask 'where sets the x and y coordinate of the block', the answer is in the definition of `blockSelection`)
```javascript
function busLayer(blockSelection, data) {
  busDot(blockSelection)
  busConnection(blockSelection, data)
}

function busDot(blockSelection) {
  blockSelection
    .append('circle')
    .attr('cx', `0.5`)
    .attr('cy', `0.5`)
    .attr('r', `0.3`)
    .style('visibility', d => d.bus ? 'visible' : 'collapse')
}

function busConnection(blockSelection, data) {
  let blockHasBus = data.filter(block => block.bus)
  const directions = [{dx: 1, dy: 0}, {dx: 0, dy: 1}, {dx: -1, dy: 0}, {dx: 0, dy: -1}]
  directions.map(direction => {
    blockSelection
      .append('line')
      .attr({
        x1: 0.5,
        y1: 0.5,
        x2: 0.5 * (1 + direction.dx),
        y2: 0.5 * (1 + direction.dy)
      })
      .style('visibility', d => {
        if (!d.bus)
          return 'collapse'
        
        let thisDirectionConnection = blockHasBus.filter(block => {
            return block.bus
              && block.x == d.x + direction.dx
              && block.y == d.y + direction.dy
          })
        
        return thisDirectionConnection.length >= 1 ? 'visible' : 'collapse'
      })
  })
}
```
Very naive implementation. Just search if the block has a neighbour both having bus stations.
#### Drawing the legends
We can reuse `blockBackgroundLayer` and `busLayer` to simplify codes.
```javascript
function descriptionLegendLayer(chart) {
  let descriptionLegendSelection = chart
    .append('g')
    .attr('transform', `translate(${0.65 * chartDimension.width}, ${0.6 * chartDimension.height})`)
    
  descriptionLayer(descriptionLegendSelection)
  legendLayer(descriptionLegendSelection)
}

function descriptionLayer(descriptionLegendSelection) {
  descriptionLegendSelection
  .append('g')
    .style('class', 'palette-Grey-500 text')
    .append('text')
    .text('Visualized Data of Nottingham')
}

function legendLayer(descriptionLegendSelection) {
  let flatMap = function(array, lambda) { 
    return Array.prototype.concat.apply([], array.map(lambda)); 
  }
  
  let legendData = flatMap(typeMapping, (type, index) => {
    return [0,1,2,3,4].map(development => {
      return {y: development, x: index, development, type: index}
    })
  })
  
  let legendSelection = descriptionLegendSelection
    .append('g')
      .attr('transform', `scale(${scale(2)}, ${scale(2)}) translate(5, 1.5)`)
      .attr('class', 'legend')
      
  let legendIconSelection = legendSelection
    .selectAll('g')
      .data(legendData)
    .enter()
      .append('g')
        .attr('transform', (d) => `translate(${d.x}, ${d.y})`)
        
  blockBackgroundLayer(legendIconSelection)
  
  legendDescriptionLayer(legendSelection)
  
  busLegendLayer(legendSelection)
}
```
(I assume no one is interested in the implementation of `busLegendLayer`)

## Credits
Many thanks to Chenjie Yu and Chen Cheng for collecting and processing the data.