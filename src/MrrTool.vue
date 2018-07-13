<template>
    <svg class="mrr-tool" ref="mrr" :data-action="action" :data-in-progress="inProgress" :width="wrapWidth" :height="wrapHeight" :viewBox="viewBox" :style="style">
        <!-- rotate guide -->
        <circle class="mrr-tool-rotate guide" v-if="mOptions.rotate" :cx="outerRadius" :cy="outerRadius" :r="outerRadius - (isMobile ? (mOptions.resizeSize) : 0)"></circle>
        <circle class="mrr-tool-rotate" v-if="isMobile && mOptions.rotate" :cx="wrapWidth / 2 - (mOptions.strokeWidth)" :cy="mOptions.resizeSize / 2 + (mOptions.strokeWidth * 2)" :r="mOptions.resizeSize" :fill="mOptions.fill" ref="rotateMobile"></circle>
        <!-- move handle -->
        <rect class="mrr-tool-move" v-if="mOptions.move" ref="move" :x="paddingLeft" :y="paddingTop" :width="value.width" :height="value.height" @contextmenu="onContextMenu"></rect>
        <!-- resize handles -->
        <g class="mrr-tool-resize-handles" v-if="!isMobile && mOptions.resize">
            <rect class="mrr-tool-resize resize-diagonal nwse top-left" :x="paddingLeft - (mOptions.resizeSize / 2)" :y="paddingTop - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeTopLeft"></rect>
            <rect class="mrr-tool-resize resize-vertical top" :x="paddingLeft + (value.width / 2) - (mOptions.resizeSize / 2)" :y="paddingTop - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeTop"></rect>
            <rect class="mrr-tool-resize resize-diagonal nesw top-right" :x="paddingLeft + value.width - (mOptions.resizeSize / 2)" :y="paddingTop - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeTopRight"></rect>
            <rect class="mrr-tool-resize resize-horizontal right" :x="paddingLeft + value.width - (mOptions.resizeSize / 2)" :y="paddingTop + (value.height / 2) - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeRight"></rect>
            <rect class="mrr-tool-resize resize-diagonal nwse bottom-right" :x="paddingLeft + value.width - (mOptions.resizeSize / 2)" :y="paddingTop + value.height - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeBottomRight"></rect>
            <rect class="mrr-tool-resize resize-vertical bottom" :x="paddingLeft + (value.width / 2) - (mOptions.resizeSize / 2)" :y="paddingTop + value.height - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeBottom"></rect>
            <rect class="mrr-tool-resize resize-diagonal nesw bottom-left" :x="paddingLeft - (mOptions.resizeSize / 2)" :y="paddingTop + value.height - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeBottomLeft"></rect>
            <rect class="mrr-tool-resize resize-horizontal left" :x="paddingLeft - (mOptions.resizeSize / 2)" :y="paddingTop + (value.height / 2) - (mOptions.resizeSize / 2)" :width="mOptions.resizeSize" :height="mOptions.resizeSize" :fill="mOptions.fill" ref="resizeLeft"></rect>
        </g>
        <!-- rotate handles -->
        <g class="mrr-tool-rotate-handles" v-if="!isMobile && mOptions.rotate">
            <circle class="mrr-tool-rotate nesw top-left" :cx="paddingLeft - mOptions.padding + mOptions.rotateSize - mOptions.strokeWidth" :cy="paddingTop - mOptions.padding + mOptions.rotateSize + mOptions.strokeWidth" :r="mOptions.rotateSize" :fill="mOptions.fill" ref="rotateTopLeft"></circle>
            <circle class="mrr-tool-rotate nwse top-right" :cx="(paddingLeft + value.width) + mOptions.padding - mOptions.rotateSize - (mOptions.strokeWidth / 2)" :cy="paddingTop - mOptions.padding + mOptions.rotateSize + (mOptions.strokeWidth / 2)" :r="mOptions.rotateSize" :fill="mOptions.fill" ref="rotateTopRight"></circle>
            <circle class="mrr-tool-rotate nesw bottom-right" :cx="(paddingLeft + value.width) + mOptions.padding - (mOptions.rotateSize + mOptions.strokeWidth)" :cy="(paddingTop + value.height) + mOptions.padding - (mOptions.rotateSize + mOptions.strokeWidth)" :r="mOptions.rotateSize" :fill="mOptions.fill" ref="rotateBottomRight"></circle>
            <circle class="mrr-tool-rotate nwse bottom-left" :cx="paddingLeft - mOptions.padding + (mOptions.rotateSize + (mOptions.strokeWidth / 2))" :cy="(paddingTop + value.height) + mOptions.padding - (mOptions.rotateSize + (mOptions.strokeWidth / 2))" :r="mOptions.rotateSize" :fill="mOptions.fill" ref="rotateBottomLeft"></circle>
        </g>
        <!-- action -->
        <transition name="fade" v-if="mOptions.action">
            <text v-if="action !== null" class="mrr-tool-action" text-anchor="middle" :x="wrapWidth / 2" :y="wrapHeight / 2" dy="7" stroke="none" :fill="mOptions.stroke">{{action}}</text>
        </transition>
    </svg>
</template>

<script>
    import Hammer from 'hammerjs';

    const _getAngle = (e, rect) => {
        const center = [
            rect.left + (rect.width / 2), // rect.left + (rect.right - rect.left) / 2
            rect.top + (rect.height / 2)
        ];
        return Math.atan2(e.clientY - center[1], e.clientX - center[0]) * 180 / Math.PI + 180;
    };

    export default {
        name: 'MRRTool',
        props: {
            value: {
                default: () => ({
                    width: 200,
                    height: 200,
                    x: 0,
                    y: 0,
                    angle: 0
                }),
                type: Object
            },
            options: {
                default: () => ({}),
                type: Object
            }
        },
        data() {
            return {
                prevAngle: 0,
                realAngle: 0,
                prevDeltaX: 0,
                prevDeltaY: 0,
                action: null,
                inProgress: false,
                rotation: false,
                managers: [],
                isMobile: (/(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|ipad|iris|kindle|Android|Silk|lge |maemo|midp|mmp|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows (ce|phone)|xda|xiino/i.test(navigator.userAgent) || /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(navigator.userAgent.substr(0,4)))
            }
        },
        computed: {
            // mOptions merged from props and default
            mOptions() {
                return Object.assign({
                    padding: 26,
                    stroke: '#00C2FF',
                    strokeWidth: 2,
                    fill: 'white',
                    resizeSize: 10,
                    rotateSize: 6,
                    move: true,
                    resize: true,
                    rotate: true,
                    action: true
                }, this.options);
            },
            // outer radius of rotate guide
            outerRadius() {
                const width = ((this.value.width / 2) + this.mOptions.padding - (this.mOptions.rotateSize + (this.mOptions.strokeWidth / 2)));
                const height = ((this.value.height / 2) + this.mOptions.padding - (this.mOptions.rotateSize + (this.mOptions.strokeWidth / 2)));
                return Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2));
            },
            // size of wrap
            wrapWidth() {
                let width;
                if (this.mOptions.rotate) {
                    width = (this.outerRadius * 2) + (this.mOptions.strokeWidth * 2);
                } else if (this.mOptions.resize) {
                    width = (this.value.width + this.mOptions.resizeSize + (this.mOptions.strokeWidth * 2));
                } else {
                    width = (this.value.width + (this.mOptions.strokeWidth * 2));
                }
                return width;
            },
            wrapHeight() {
                let height;
                if (this.mOptions.rotate) {
                    height = (this.outerRadius * 2) + (this.mOptions.strokeWidth * 2);
                } else if (this.mOptions.resize) {
                    height = (this.value.height + this.mOptions.resizeSize + (this.mOptions.strokeWidth * 2));
                } else {
                    height = (this.value.height + (this.mOptions.strokeWidth * 2));
                }
                return height;
            },
            // padding left
            paddingLeft() {
                return ((this.wrapWidth - this.value.width) / 2);
            },
            // padding top
            paddingTop() {
                return ((this.wrapHeight - this.value.height) / 2);
            },
            // viebBox attribute for svg element
            viewBox() {
                return this.mOptions.rotate ? `-2 -2 ${this.wrapWidth}, ${this.wrapHeight}` : `0 0 ${this.wrapWidth}, ${this.wrapHeight}`;
            },
            // svg element style
            style() {
                return {
                    top: `-${this.paddingTop}px`,
                    left: `-${this.paddingLeft}px`,
                    stroke: this.mOptions.stroke,
                    strokeWidth: `${this.mOptions.strokeWidth}px`,
                    fill: 'transparent'
                }
            }
        },
        mounted() {
            // desktop listeners
            if (!this.isMobile) {
                // add resize listeners
                if (this.mOptions.resize) {
                    this.addPanListener(this.$refs.resizeTopLeft, {
                        width: true,
                        height: true,
                        x: true,
                        y: true,
                        angle: false
                    }, {invertX: true, invertY: true}, 'resize');
                    this.addPanListener(this.$refs.resizeTop, {
                        width: false,
                        height: true,
                        x: false,
                        y: true,
                        angle: false
                    }, {invertX: false, invertY: true}, 'resize');
                    this.addPanListener(this.$refs.resizeTopRight, {
                        width: true,
                        height: true,
                        x: false,
                        y: true,
                        angle: false
                    }, {invertX: false, invertY: true}, 'resize');
                    this.addPanListener(this.$refs.resizeRight, {
                        width: true,
                        height: false,
                        x: false,
                        y: false,
                        angle: false
                    }, {invertX: false, invertY: false}, 'resize');
                    this.addPanListener(this.$refs.resizeBottomRight, {
                        width: true,
                        height: true,
                        x: false,
                        y: false,
                        angle: false
                    }, {invertX: false, invertY: false}, 'resize');
                    this.addPanListener(this.$refs.resizeBottom, {
                        width: false,
                        height: true,
                        x: false,
                        y: false,
                        angle: false
                    }, {invertX: false, invertY: false}, 'resize');
                    this.addPanListener(this.$refs.resizeBottomLeft, {
                        width: true,
                        height: true,
                        x: true,
                        y: false,
                        angle: false
                    }, {invertX: true, invertY: false}, 'resize');
                    this.addPanListener(this.$refs.resizeLeft, {
                        width: true,
                        height: false,
                        x: true,
                        y: false,
                        angle: false
                    }, {invertX: true, invertY: false}, 'resize');
                }
                // add rotate listener
                if (this.mOptions.rotate) {
                    this.addPanListener(this.$refs.rotateTopLeft, {
                        width: false,
                        height: false,
                        x: false,
                        y: false,
                        angle: true
                    }, {}, 'rotate');
                    this.addPanListener(this.$refs.rotateTopRight, {
                        width: false,
                        height: false,
                        x: false,
                        y: false,
                        angle: true
                    }, {}, 'rotate');
                    this.addPanListener(this.$refs.rotateBottomLeft, {
                        width: false,
                        height: false,
                        x: false,
                        y: false,
                        angle: true
                    }, {}, 'rotate');
                    this.addPanListener(this.$refs.rotateBottomRight, {
                        width: false,
                        height: false,
                        x: false,
                        y: false,
                        angle: true
                    }, {}, 'rotate');
                }
                // add move listener
                if (this.mOptions.move) {
                    this.addPanListener(this.$refs.move, {
                        width: false,
                        height: false,
                        x: true,
                        y: true,
                        angle: false
                    }, {invertX: false, invertY: false}, 'move');
                }
                // register hover events
                this.managers.forEach(manager => {
                    manager.element.addEventListener('mouseenter', function () { this.hoverStart(manager.name); }.bind(this));
                    manager.element.addEventListener('mouseleave', function () { this.hoverEnd(manager.name); }.bind(this));
                });
            }
            // mobile listeners
            else {
                // register move, resize listeners on move element
                const recognizers = [];
                if (this.mOptions.move) recognizers.push([Hammer.Pan, { direction: Hammer.DIRECTION_ALL }]);
                if (this.mOptions.resize) recognizers.push([Hammer.Pinch]);
                recognizers.push([Hammer.Press]);
                const moveResizeManager = new Hammer.Manager(this.$refs.move, { recognizers });
                // move
                if (this.mOptions.move) {
                    moveResizeManager.on('panstart', function () {
                        this.inProgress = true;
                        this.action = 'move';
                    }.bind(this));
                    moveResizeManager.on('panmove', function (e) {
                        // calculate actual delta
                        const deltaX = e.srcEvent.movementX;
                        const deltaY = e.srcEvent.movementY;

                        // prepare output
                        const newValue = {
                            width: this.value.width,
                            height: this.value.height,
                            x: this.value.x + deltaX,
                            y: this.value.y + deltaY,
                            angle: this.value.angle
                        };
                        // emit event
                        this.$emit('input', newValue);
                    }.bind(this));
                    moveResizeManager.on('panend', function () {
                        this.inProgress = false;
                        this.action = null;
                        this.$emit('changed');
                    }.bind(this));
                }
                // resize
                if (this.mOptions.resize) {
                    moveResizeManager.on('pinchstart', function (e) {
                        this.inProgress = true;
                        this.action = 'resize';
                        this.prevDeltaX = e.deltaX;
                        this.prevDeltaY = e.deltaY;
                    }.bind(this));
                    moveResizeManager.on('pinch', function (e) {
                        const newValue = {
                            width: this.value.width + (e.deltaX - this.prevDeltaX),
                            height: this.value.height + (e.deltaY - this.prevDeltaY),
                            x: this.value.x,
                            y: this.value.y,
                            angle: this.value.angle
                        };
                        this.prevDeltaX = e.deltaX;
                        this.prevDeltaY = e.deltaY;
                        // emit event
                        this.$emit('input', newValue);
                    }.bind(this));
                    moveResizeManager.on('pinchend', function () {
                        this.inProgress = false;
                        this.action = null;
                        this.prevDeltaX = 0;
                        this.prevDeltaY = 0;
                        this.$emit('changed');
                    }.bind(this));
                }
                // press
                moveResizeManager.on('press', function (e) {
                    this.onContextMenu(e.srcEvent);
                }.bind(this));
                // add manager to array
                this.managers.push(moveResizeManager);
                // register rotate listener
                if (this.mOptions.rotate) {
                    this.addPanListener(this.$refs.rotateMobile, {width: false,  height: false,  x: false,   y: false,   angle: true}, {}, 'rotate');
                }
            }
        },
        beforeDestroy() {
            // remove listeners
            this.managers.forEach(m => {this.removeListeners(m)});
        },
        methods: {
            addPanListener(element, changes, mOptions, name) {
                // create hammer manager
                const manager = new Hammer.Manager(element, {recognizers: [[Hammer.Pan]]});
                // register event listeners
                manager.on('panstart', function (e) {
                    this.inProgress = true;
                    this.action = name;
                    this.prevAngle = changes.angle ? _getAngle(e.srcEvent, this.$refs.mrr.getBoundingClientRect()) : 0;
                }.bind(this));
                manager.on('panmove', function (e) {
                    // calculate actual delta
                    const deltaX = e.srcEvent.movementX;
                    const deltaY = e.srcEvent.movementY;
                    let deltaAngle = 0;
                    let newAngle = this.value.angle;
                    // calculate change in angle
                    if (changes.angle) {
                        const angle = _getAngle(e.srcEvent, this.$refs.mrr.getBoundingClientRect());
                        deltaAngle = angle - this.prevAngle;
                        this.prevAngle = angle;
                        if (e.srcEvent.shiftKey) {
                            let tmpAngle = Math.round(this.realAngle + deltaAngle);
                            this.realAngle = tmpAngle;
                            if (tmpAngle === (Math.round(tmpAngle / 10) * 10)) {
                                newAngle = tmpAngle;
                            } else {
                                newAngle = this.value.angle;
                            }
                        } else {
                            newAngle = Math.round(this.value.angle + deltaAngle);
                            this.realAngle = newAngle;
                        }
                    }
                    
                    // prepare output
                    const newValue = {
                        width: changes.width ? (mOptions.invertX ? this.value.width - deltaX : this.value.width + deltaX) : this.value.width,
                        height: changes.height ? (mOptions.invertY ? this.value.height - deltaY : this.value.height + deltaY) : this.value.height,
                        x: changes.x ? this.value.x + deltaX : this.value.x,
                        y: changes.y ? this.value.y + deltaY : this.value.y,
                        angle: newAngle
                    };
                    // emit event
                    this.$emit('input', newValue);
                }.bind(this));
                manager.on('panend', function () {
                    this.inProgress = false;
                    this.action = null;
                    this.$emit('changed');
                }.bind(this));
                // push name to manager
                manager.name = name;
                // add manager to array
                this.managers.push(manager);
            },
            removeListeners(manager) {
                for (let handler of manager.handlers) {
                    manager.remove(handler);
                }
                manager.destroy();
            },
            hoverStart(name) {
                if (!this.inProgress && this.action !== name) this.action = name;
            },
            hoverEnd() {
                if (!this.inProgress) this.action = null;
            },
            onContextMenu(e) {
                if (this.$listeners.contextmenu) {
                    e.preventDefault();
                    this.$emit('contextmenu', e);
                }
            }
        }
    }
</script>

<style scoped lang="scss">
    $opacity-transparent: 0;
    $opacity-light: .2;
    $opacity-medium: .4;
    $opacity-hard: .8;
    $duration: .25s;
    .mrr-tool {
        position: absolute;
        display: block;
        stroke: aqua;
        touch-action: none;
    }
    .mrr-tool-move {
        cursor: move;
        transition: opacity $duration;
    }
    .mrr-tool-resize {
        transition: opacity $duration;
        &.resize-diagonal {
            &.nwse {
                cursor: nwse-resize;
            }
            &.nesw {
                cursor: nesw-resize;
            }
        }
        &.resize-vertical {
            cursor: ns-resize;
        }
        &.resize-horizontal {
            cursor: ew-resize;
        }
    }
    .mrr-tool-rotate {
        cursor: move;
        transition: opacity $duration;
        &.guide {
            opacity: $opacity-light;
            stroke-dasharray: 2px, 6px;
            transition: stroke-dasharray $duration, opacity $duration;
        }
    }
    .mrr-tool-action {
        text-transform: uppercase;
        pointer-events: none;
    }
    // special cases
    .mrr-tool[data-action="rotate"] {
        .mrr-tool-rotate.guide { 
            opacity: $opacity-medium;
            stroke-dasharray: 2px, 2px;
        }
        .mrr-tool-move, .mrr-tool-resize {
            opacity: $opacity-medium;
        }
        &[data-in-progress="true"] {
            .mrr-tool-rotate.guide {
                opacity: $opacity-hard;
            }
            .mrr-tool-move {
                opacity: $opacity-light;
            }
            .mrr-tool-resize {
                opacity: $opacity-transparent;
            }
        }
    }
    .mrr-tool[data-action="move"] {
        .mrr-tool-rotate { 
            opacity: $opacity-medium;
        }
        .mrr-tool-resize {
            opacity: $opacity-medium;
        }
        &[data-in-progress="true"] {
            .mrr-tool-rotate {
                &:not(.guide) {
                    opacity: $opacity-transparent;
                }
            }
            .mrr-tool-resize {
                opacity: $opacity-transparent;
            }
        }
    }
    .mrr-tool[data-action="resize"] {
        .mrr-tool-rotate { 
            opacity: $opacity-medium;
        }
        .mrr-tool-move {
            opacity: $opacity-medium;
        }
        &[data-in-progress="true"] {
            .mrr-tool-rotate {
                opacity: $opacity-transparent;
            }
            .mrr-tool-move {
                opacity: $opacity-light;
            }
        }
    }
    // transition
    .fade-enter-active, .fade-leave-active {
        transition: opacity ($duration * 2);
    }
    .fade-enter, .fade-leave-to {
        opacity: 0;
    }
</style>