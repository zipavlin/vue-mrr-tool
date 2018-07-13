# Vue move-resize-rotate component

Move-resize-rotate component for [Vue](https://vuejs.org/) framework. Component acts as a tool for editing other (parent) component.  
It uses [hammerjs](https://hammerjs.github.io/) and supports desktop and mobile. Tested on chrome, firefox, edge and chrome mobile.

## Install

```javascript
npm install @zipavlin/mrr-tool
```

```html
<script src="//unpkg.com/@zipavlin/vue-mrr-tool/dist/MrrTool.umd.min.js">
```

## Use

Include component in your Vue app and bind v-model or :value + @input attributes.  

Keep in mind that this component is supposed to be used as a tool, which means that it does not store value properties (width, height, x, y, angle) internaly, but expects to get them from external source (parent component/vuex store).  
  
The easiest way to use this tool is to bind component's output object to vue style object.

```html
<template>
    <div id="app">
        <div :style="style">
            <mrr-tool v-model="mrr"></mrr-tool>
        </div>
    </div>
</template>

<script>
    import MrrTool from '@zipavlin/vue-mrr-tool'

    export default {
        name: "App",
        components: { MrrTool },
        data() {
            return {
                mrr: {
                    width: 200,
                    height: 100,
                    x: 0,
                    y: 0,
                    angle: 0
                }
            }
        },
        computed: {
            style() {
                return {
                    position: 'absolute',
                    top: this.mrr.y + 'px',
                    left: this.mrr.x + 'px',
                    width: this.mrr.width + 'px',
                    height: this.mrr.height + 'px',
                    transform: `rotate(${this.mrr.angle}deg)`
                };
            }
        }
    }
</script>
```

## Props

### Value
Type __Object__
Component supports binding with __v-model__ or __:value__ + __@input__ combination.

| value         | default       | description                                |
| ------------- |:-------------:| ------------------------------------------:|
| width         | 200           | width (in px)                              |
| height        | 200           | height (in px)                             |
| x             | 0             | x/left position (in px)                    |
| y             | 0             | y/top position (in px)                     |
| angle         | 0             | angle (in deg)                             |

### Options
Type __Object__ 

| option        | default       | description                                |
| ------------- |:-------------:| ------------------------------------------:|
| padding       | 26            | padding between resize and rotate handles  |
| stroke        | '#00C2FF'     | stroke color                               |
| strokeWidth   | 2             | stroke width (in px)                       |
| fill          | 'white'       | resize and rotate handles fill color       |
| resizeSize    | 10            | size of resize handles                     |
| rotateSize    | 6             | size of rotate handles                     |
| move          | true          | should tool allow move edit                |
| resize        | true          | should tool allow resize edit              |
| rotate        | true          | should tool allow rotate edit              |
| action        | true          | should tool display action                 |

## Events

### input

Triggered on every move/resize/rotate event (not just at the end).

### change

Triggered when edit is done with no parameters passed. This can be used for saving a history state when using vuex history.

### contextmenu

Triggered on right click (mouse) or long touch (touch) on _move_ element with event object passed as only parameter.