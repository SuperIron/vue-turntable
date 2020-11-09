<template>
    <view class="turntable">
        <view
            class="turntable__item"
            :class="[curIdx === idx ? 'turntable__item--active' : null]"
            :style="{ gridArea: gridAreas[idx] }"
            v-for="(item, idx) in options"
            :key="'TURNTABLE_' + idx"
        >
            索引：{{ idx }}
        </view>
        <view class="turntable__raffle" @click="raffle">抽奖</view>
    </view>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";

type Direction = "up" | "down" | "left" | "right";
type Rotation = "clockwise" | "anticlockwise";

const ROW = 3; // 行数
const COL = 3; // 列数
const MAX_DELAY = 300; // 最大跳动间隔
const MIN_DELAY = 50; // 最小跳动间隔
const ROLL = 3; // 跳动循环圈数
const CLOCKWISE_DIRECTIONS: Direction[] = ["right", "down", "left", "up"]; // 顺时针转向的路径
const ANTICLOCKWISE_DIRECTIONS: Direction[] = ["down", "right", "up", "left"]; // 逆时针转向的路径

@Component
export default class extends Vue {
    /**
     * 奖品列表
     */
    @Prop({ default: () => [] }) options: any[] = [];
    /**
     * 跳动方向，默认顺时针
     */
    @Prop({ default: "clockwise" }) rotation: Rotation = "clockwise";
    /**
     * 开始抽奖前，在回调函数中返回中奖索引
     */
    @Prop({ default: () => {}, required: true }) beforeRaffle!: () => number;

    private timer!: number;
    private curIdx: number = 0; // 当前激活的索引
    private prizeIdx: number = 0; // 中奖索引
    private count: number = 0; // 跳动次数
    private isPlaying: boolean = false;

    /**
     * 跳动间隔
     */
    private get delay() {
        if (this.count > 10) {
            return MIN_DELAY;
        }
        return MAX_DELAY - ((MAX_DELAY - MIN_DELAY) / 10) * this.count;
    }

    /**
     * 跳动路径
     */
    private get directions() {
        return this.rotation === "clockwise"
            ? CLOCKWISE_DIRECTIONS
            : ANTICLOCKWISE_DIRECTIONS;
    }

    /**
     * 奖品grid-area值
     */
    private get gridAreas() {
        let x = 1;
        let y = 1;
        return this.directions.reduce((pre, item) => {
            switch (item) {
                case "up":
                    while (y > 1) {
                        pre.push(`${y}/${x}`);
                        y--;
                    }
                    y = y < 1 ? 1 : y;
                    break;
                case "down":
                    while (y < ROW) {
                        pre.push(`${y}/${x}`);
                        y++;
                    }
                    y = y > COL ? COL : y;
                    break;
                case "left":
                    while (x > 1) {
                        pre.push(`${y}/${x}`);
                        x--;
                    }
                    x = x < 1 ? 1 : x;
                    break;
                case "right":
                    while (x < COL) {
                        pre.push(`${y}/${x}`);
                        x++;
                    }
                    x = x > ROW ? ROW : x;
                    break;
            }
            return pre;
        }, [] as string[]);
    }

    /**
     * 执行抽奖动画
     */
    private animate() {
        if (this.curIdx >= this.options.length - 1) {
            this.curIdx = 0;
        } else {
            this.curIdx++;
        }
        this.count--;
        if (this.count <= 0) {
            this.ended();
            return;
        }
        this.timer = setTimeout(() => {
            this.animate();
        }, this.delay);
    }

    /**
     * 开始抽奖
     */
    private raffle() {
        if (this.isPlaying) {
            uni.showToast({
                title: "正在抽奖...",
                icon: "none"
            });
            return;
        }
        this.isPlaying = true;
        this.prizeIdx = this.beforeRaffle.call(this.$parent);
        this.setCount();
        this.animate();
    }

    /**
     * 抽奖结束
     */
    private ended() {
        this.isPlaying = false;
        this.$emit("on-ended", this.curIdx);
    }

    /**
     * 设置跳动次数
     */
    private setCount() {
        const count = this.prizeIdx - this.curIdx;
        const len = this.options.length;
        this.count = ROLL * len + (count > 0 ? count : count + len);
    }

    destroyed() {
        clearTimeout(this.timer);
    }
}
</script>

<style lang="less" scoped>
.turntable {
    width: 630rpx;
    height: 630rpx;
    margin: 24rpx auto 76rpx;
    padding: 21rpx;
    background: rgba(255, 255, 255, 0.3);
    display: grid;
    grid-template-columns: 184rpx 184rpx 184rpx;
    grid-template-rows: 184rpx 184rpx 184rpx;
    grid-row-gap: 18rpx;
    grid-column-gap: 18rpx;

    &__raffle {
        background: #ffda44;
        grid-area: ~"2 / 2";
    }

    &__item {
        background: #fff;
        opacity: 0.3;

        &--active {
            opacity: 1;
        }
    }
}
</style>
