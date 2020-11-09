# vue-turntable

## 基于`uni-app`+`ts`+`less`的九宫格抽奖组件

## 配置

| 属性      | 说明                                                                        | 类型     | 必填 | 默认值      |
| --------- | --------------------------------------------------------------------------- | -------- | ---- | ----------- |
| options     | 奖品列表                                                            | Array   | 是   | -           |
| rotation     |跳动方向`clockwise` `anticlockwise`         | String   | 否   | clockwise |
| beforeRaffle   |开始抽奖前的钩子，在回调函数中返回中奖索引                                                       | Function   | 是   | -        |
