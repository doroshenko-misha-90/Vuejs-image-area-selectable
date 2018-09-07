<template>
    <div class="img-area">
        <div class="img-area-backface"></div>
        <img ref="img" :src="src" alt="">
        <div class="img-area-box" :style="{
					'width': `${area.w}px`,
					'height': `${area.h}px`,
					'transform': `translate3d(${areaOffsetX}px,  ${areaOffsetY}px, ${0})`
					}">
            <div class="img-area-view-box">
                <img :src="src" alt="" :style="{
                    width: `${trueWidth}px`,
                    height: `${trueHeight}px`,
                    'transform': `scale(${scale}, ${scale}) translate3d(${(x-areaOffsetX) / scale}px,  ${(y-areaOffsetY) / scale}px, ${0})`
                }">
            </div>
            <div class="img-area-face-move"
                 @mousedown="moveAreaStart"
                 @touchstart="moveAreaStart"
            ></div>

            <span class="img-area-line line-w" @mousedown="changeAreaStart($event, DIRECTION_TOP)"
                  @touchstart="changeAreaStart($event, DIRECTION_TOP)"></span>
            <span class="img-area-line line-a" @mousedown="changeAreaStart($event, DIRECTION_LEFT)"
                  @touchstart="changeAreaStart($event, DIRECTION_LEFT)"></span>
            <span class="img-area-line line-s" @mousedown="changeAreaStart($event, DIRECTION_BOTTOM)"
                  @touchstart="changeAreaStart($event, DIRECTION_BOTTOM)"></span>
            <span class="img-area-line line-d" @mousedown="changeAreaStart($event, DIRECTION_RIGHT)"
                  @touchstart="changeAreaStart($event, DIRECTION_RIGHT)"></span>
            <span class="img-area-point point-left-top"  @mousedown="changeAreaStart($event, DIRECTION_LEFT_TOP)"
                  @touchstart="changeAreaStart($event, DIRECTION_LEFT_TOP)"></span>
            <span class="img-area-point point-left-bottom"  @mousedown="changeAreaStart($event, DIRECTION_LEFT_BOTTOM)"
                  @touchstart="changeAreaStart($event, DIRECTION_LEFT_BOTTOM)"></span>
            <span class="img-area-point point-right-top"  @mousedown="changeAreaStart($event, DIRECTION_RIGHT_TOP)"
                  @touchstart="changeAreaStart($event, DIRECTION_RIGHT_TOP)"></span>
            <span class="img-area-point point-right-bottom"  @mousedown="changeAreaStart($event, DIRECTION_RIGHT_BOTTOM)"
                  @touchstart="changeAreaStart($event, DIRECTION_RIGHT_BOTTOM)"></span>
        </div>
    </div>
</template>

<script>
    export default {
        props: {
            src: {
                required: true,
                type: String
            },
            value: {
                required: true,
                type: Object,
                validator: value =>  {
                return !_.filter(['x', 'y', 'w', 'h'], prop => {
                    return !value.hasOwnProperty(prop);
    }).length;
    }
    }
    },

    data() {
        return {
            w: 0,
            h: 0,
            x: 0,
            y: 0,
            scale: 1,

            trueWidth: 0,
            trueHeight: 0,

            area: {
                w: 0,
                h: 0,
                x: 0,
                y: 0,
            },

            DIRECTION_LEFT: 0,
            DIRECTION_RIGHT: 1,
            DIRECTION_TOP: 2,
            DIRECTION_BOTTOM: 3,
            DIRECTION_LEFT_TOP: 4,
            DIRECTION_LEFT_BOTTOM: 5,
            DIRECTION_RIGHT_TOP: 6,
            DIRECTION_RIGHT_BOTTOM: 7,

            oldArea: {},

            difference: {
                w: 0,
                h: 0
            },

            areaOffsetX: 0,
            areaOffsetY: 0,

            oldAreaOffsetX: 0,
            oldAreaOffsetY: 0,

            minX: 0,
            minY: 0,
            changeAreaToDirection: null

        }
    },

    watch: {
        src() {
            this.reload();
        },
        value() {
            this.setAreaDefault();
        }
    },

    methods: {
        setAreaDefault() {

            this.setAreaAttributeDefault('w');
            this.setAreaAttributeDefault('h');

            this.setAreaOffsetDefault('x');
            this.setAreaOffsetDefault('y');

        },
        setValue() {
            this.$emit('input', {
                w: this.area.w / this.scale,
                h: this.area.h / this.scale,
                y: this.areaOffsetY / this.scale,
                x: this.areaOffsetX / this.scale
            });
        },
        setAreaOffsetDefault(attribute) {
            this[`areaOffset${_.upperCase(attribute)}`] = this.value[attribute] * this.scale;
        },
        setAreaAttributeDefault(attribute) {
            this.area[attribute] = this.value[attribute] ? this.value[attribute] * this.scale : this[attribute];
        },

        reload() {
            let img = new Image();
            img.onload = () => {

                this.w = ~~window
                    .getComputedStyle(this.$refs.img)
                    .width.replace("px", "");
                this.h = ~~window
                    .getComputedStyle(this.$refs.img)
                    .height.replace("px", "");

                this.trueWidth = img.width;
                this.trueHeight = img.height;

                if (this.trueWidth > this.w) {
                    this.scale = this.w / this.trueWidth;
                }

                if (this.trueHeight * this.scale > this.h) {
                    this.scale = this.h / this.trueHeight;
                }

                this.$nextTick(() => {
                    this.x = this.calculateScalePosition(this.trueWidth, this.w);
                this.y = this.calculateScalePosition(this.trueHeight, this.h);
                this.setAreaDefault();
                this.$emit("imgLoad", "success");
            });

            };

            img.onerror = () => {
                this.$emit("imgLoad", "error");
            };
            img.src = this.src;
        },

        calculateScalePosition(original, current) {
            return -(original - original * this.scale) / 2 +
                (current - original * this.scale) / 2;
        },

        getMoveListeners() {
            return [
                {
                    events: ['mousemove', 'touchmove'],
                    func: this.moveAreaNow
                },
                {
                    events: ['mouseup', 'touchend'],
                    func: this.moveAreaEnd
                }
            ]
        },

        moveAreaStart(e) {
            e.preventDefault();

            this.listenerController(this.getMoveListeners());

            this.area.x = this.getCoordByEvent(e, 'X') - this.areaOffsetX;
            this.area.y = this.getCoordByEvent(e, 'Y') - this.areaOffsetY;
        },

        moveAreaNow(e, isMove) {
            let nowX = this.getCoordByEvent(e, 'X'),
                nowY = this.getCoordByEvent(e, 'Y');
            if (e) {
                e.preventDefault();
            }
            this.$nextTick(() => {
                let fw = isMove ? this.areaOffsetX : ~~(nowX - this.area.x),
                fh = isMove ? this.areaOffsetY : ~~(nowY - this.area.y);

            this.areaOffsetX = this.getOffsetCoord(fw, this.area.w, this.w);
            this.areaOffsetY = this.getOffsetCoord(fh, this.area.h, this.h);
        });
        },

        moveAreaEnd() {
            this.listenerController(this.getMoveListeners(), 'removeEventListener');
            this.setValue();
        },

        changeAreaStart(e, direction) {
            this.changeAreaToDirection = direction;

            this.area.x = this.getCoordByEvent(e, 'X');
            this.area.y = this.getCoordByEvent(e, 'Y');
            this.oldArea = Object.assign({}, this.area);
            this.oldAreaOffsetX = this.areaOffsetX;
            this.oldAreaOffsetY = this.areaOffsetY;
            this.listenerController(this.getChangeAreaListeners());
        },

        changeAreaEnd(e) {
            this.listenerController(this.getChangeAreaListeners(), 'removeEventListener');
            this.setValue();
        },

        getChangeAreaListeners() {
            return [
                {
                    events: ['mousemove', 'touchmove'],
                    func: this.changeAreaNow
                },
                {
                    events: ['mouseup', 'touchend'],
                    func: this.changeAreaEnd
                }
            ]
        },

        changeAreaNow(e) {

            let nowX = this.getCoordByEvent(e, 'X'),
                nowY = this.getCoordByEvent(e, 'Y');

            this.$nextTick(() => {
                this.difference.w = ~~(nowX - this.area.x);
            this.difference.h = ~~(nowY - this.area.y);

            switch (this.changeAreaToDirection) {
                case this.DIRECTION_LEFT:
                    this.changeAreaWidthLeft();
                    break;
                case this.DIRECTION_RIGHT:
                    this.changeAreaWidthRight();
                    break;
                case this.DIRECTION_TOP:
                    this.changeAreaHeightTop();
                    break;
                case this.DIRECTION_BOTTOM:
                    this.changeAreaHeightBottom();
                    break;
                case this.DIRECTION_LEFT_TOP:
                    this.changeAreaWidthLeft();
                    this.changeAreaHeightTop();
                    break;
                case this.DIRECTION_LEFT_BOTTOM:
                    this.changeAreaWidthLeft();
                    this.changeAreaHeightBottom();
                    break;
                case this.DIRECTION_RIGHT_TOP:
                    this.changeAreaWidthRight();
                    this.changeAreaHeightTop();
                    break;
                case this.DIRECTION_RIGHT_BOTTOM:
                    this.changeAreaWidthRight();
                    this.changeAreaHeightBottom();
                    break;
            }

        });
        },

        changeAreaWidthLeft() {
            this.setAreaAttribute('w', 'X');
        },

        changeAreaHeightTop() {
            this.setAreaAttribute('h', 'Y');
        },

        changeAreaWidthRight() {
            this.setAreaAttribute('w', 'X', 'Plus');
        },

        changeAreaHeightBottom() {
            this.setAreaAttribute('h', 'Y', 'Plus');
        },

        setAreaAttribute(attribute, axisAttribute, funcSuffix = 'Minus') {
            let old = this.oldArea[attribute],
                offsetAttribute = `areaOffset${axisAttribute}`,
                oldOffset = this[`oldAreaOffset${axisAttribute}`],
                min = this[`min${axisAttribute}`],
                diff = this.difference[attribute],
                wrapper = this[attribute];

            this[`setAreaAttribute${funcSuffix}`](attribute, offsetAttribute, old, oldOffset, diff, wrapper, min);
        },

        setAreaAttributeMinus(attribute, offsetAttribute, old, oldOffset, diff, wrapper, min) {
            if (old - diff > 0) {
                let isOffset = wrapper - oldOffset - diff <= wrapper - min;
                this.area[attribute] = isOffset ? old - diff : old + oldOffset - min;
                this[offsetAttribute] = isOffset ? oldOffset + diff : min;
            } else {
                this.area[attribute] = Math.abs(diff) + oldOffset <= wrapper
                    ? Math.abs(diff) - old
                    : wrapper - old - oldOffset;
                this[offsetAttribute] = oldOffset + old;
            }
        },

        setAreaAttributePlus(attribute, offsetAttribute, old, oldOffset, diff, wrapper, min) {
            if (old + diff > 0) {
                this.area[attribute] =
                    old + diff + oldOffset <= wrapper ? old + diff : wrapper - oldOffset;
                this[offsetAttribute] = oldOffset;
            } else {
                let isOffsetRight = wrapper - oldOffset + Math.abs(diff + old) <= wrapper - min;
                this.area[attribute] = isOffsetRight ? Math.abs(diff + old) : oldOffset - wrapper;
                this[offsetAttribute] = isOffsetRight ? oldOffset - Math.abs(diff + old) : min;
            }
        },

        listenerController(listeners, method = 'addEventListener') {
            _.forEach(listeners, listener => {
                _.forEach(listener.events, event => {
                window[method](event, listener.func)
            });
        })
        },


        getCoordByEvent(e, coord) {
            if (!e) return 0;
            let key = [`client${coord}`];
            return e[key] ? e[key] : e.touches[0][key];
        },

        getOffsetCoord(n, area, full) {
            if (n <= 1) {
                return 1;
            } else if (~~(n + area) > full) {
                return full - area - 1;
            }
            return n
        }

    },
    mounted() {
        this.reload();
    }
    }
</script>

<style scoped>
    .img-area {
        width: 100%;
        height: 100%;
        position: relative;
    }

    .img-area-backface,
    .img-area-box,
    .img-area-face-move {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        user-select: none;
    }

    .img-area-box {
        overflow: hidden;
        z-index: 2;
    }

    .img-area-face-move {
        width: 100%;
        height: 100%;
        background: rgba(255, 255, 255, .2);
        border: 4px solid #fff;
        cursor: move;
    }

    .img-area-view-box {
        width: 100%;
        height: 100%;
        position: relative;
    }

    .img-area-backface {
        width: 100%;
        height: 100%;
        background-color: rgba(21, 21, 21, 0.3);
        z-index: 1;
    }

    .img-area-line {
        position: absolute;
        display: block;
        width: 100%;
        height: 100%;
        opacity: 0.1;
    }

    .line-w {
        top: -4px;
        left: 0;
        height: 8px;
        cursor: n-resize;
    }

    .line-a {
        top: 0;
        left: -4px;
        width: 8px;
        cursor: w-resize;
    }

    .line-s {
        bottom: -4px;
        left: 0;
        height: 8px;
        cursor: s-resize;
    }

    .line-d {
        top: 0;
        right: -4px;
        width: 8px;
        cursor: e-resize;
    }

    .img-area-point {
        position: absolute;
        display: block;
        width: 14px;
        height: 14px;
    }

    .point-left-top{
        left: -7px;
        top: -7px;
        cursor: nw-resize;
    }

    .point-left-bottom{
        left: -7px;
        bottom: -7px;
        cursor: ne-resize;
    }

    .point-right-top{
        right: -7px;
        top: -7px;
        cursor: sw-resize;
    }

    .point-right-bottom{
        right: -7px;
        bottom: -7px;
        cursor: se-resize;
    }

</style>