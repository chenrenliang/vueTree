/**
* Created by 王冬 on 2017/9/20.
* QQ: 20004604
* weChat: qq20004604
*/

<template>
  <div :style="rootStyle" class="root">
    <div class="topNode" :style="topNodeStyle" lv="topNodeStyle" ref="parent">
      <div class="content" :style="contentStyle">
        <div class="btn-box" :style="btnBoxStyle" ref="btnBox">
          <!-- 点击下拉图标组 -->
          <template v-if="settings.openBtn.enabled">
            <!-- 默认图标 -->
            <template v-if="!settings.openBtn.customIcon">
              <span class="btn-span" :style="dropBtnStyle" @click="hideOrShow">{{isOpened ? '－' : '＋'}}</span>
            </template>

            <!-- 自定义图标 -->
            <template v-if="settings.openBtn.customIcon">
              <!-- 有子节点时 -->
              <template v-if="data.children">
                <!-- 自定义打开 -->
                <img v-if="isOpened" :src="settings.openBtn.openedIcon" alt="" :style="dropBtnStyle"
                     @click="hideOrShow()">
                <!-- 自定义关闭 -->
                <img v-if="!isOpened" :src="settings.openBtn.closedIcon" alt=""
                     :style="dropBtnStyle"
                     @click="hideOrShow()">
              </template>
              <!-- 无子节点时 -->
              <template v-if="!data.children">
                <!-- 关闭自定义图标（显示空白） -->
                <template v-if="!settings.openBtn.noChildIcon.enabled">
                  <img class="openbtn-nochildren" :src="settings.openBtn.noChildIcon.defaultImage" alt=""
                       :style="dropBtnStyle">
                </template>
                <!-- 打开自定义图标 -->
                <template v-if="settings.openBtn.noChildIcon.enabled">
                  <!-- 统一使用指定自定义图标 -->
                  <img v-if="settings.openBtn.noChildIcon.useDefaultImage"
                       class="openbtn-nochildren" :src="settings.openBtn.noChildIcon.defaultImage" alt=""
                       :style="btnBoxStyle">
                  <!-- 使用用户自定义图标 -->
                  <template v-if="!settings.openBtn.noChildIcon.useDefaultImage">
                    <!-- 有用户自定义图标时，使用用户自定义图标 -->
                    <img v-if="data[settings.openBtn.noChildIcon.customIconKey]" class="openbtn-nochildren"
                         :src="data[settings.openBtn.noChildIcon.customIconKey]" alt=""
                         :style="btnBoxStyle">
                    <!-- 没有用户自定义图标时，使用用户默认自定义图标 -->
                    <img v-if="!data[settings.openBtn.noChildIcon.customIconKey]" class="openbtn-nochildren"
                         :src="settings.openBtn.noChildIcon.withOutCustomIconKey" alt=""
                         :style="btnBoxStyle">
                  </template>
                </template>
              </template>
            </template>
          </template>

          <!-- 点击选中图标组 -->
          <template v-if="settings.checkBox.enabled">
            <img v-if="checkedStatus===2" :src="settings.checkBox.checkedIcon" alt=""
                 :style="checkBoxStyle" @click="changeSelectStatus()">
            <img v-if="checkedStatus===1" :src="settings.checkBox.halfCheckedIcon" alt=""
                 :style="checkBoxStyle" @click="changeSelectStatus()">
            <img v-if="checkedStatus===0" :src="settings.checkBox.uncheckedIcon" alt=""
                 :style="checkBoxStyle" @click="changeSelectStatus()">
          </template>
        </div>
        <span class="text-box" :style="textBoxStyle" @mouseover="mouseoverTransition" @mouseout="mouseoutTransition">
              <span class="text" :class="textClass" :style="textStyle"
                    ref="textSpan"
                    @click="clickEvent"
                    @dblclick="debouleClickEvent"
                    @mouseover="mouseoverEvent"
                    @mouseout="mouseoutEvent">
                <!--{{level}}：-->
                {{data.name}}
                <span :class="{underline:settings.underLine}"></span>
              </span>
          </span>
      </div>
    </div>

    <transition
      v-on:before-enter="beforeEnter"
      v-on:enter="enter"
      v-on:leave="leave">
      <div v-if="isOpened" style="transform-origin: 50% 0;">
        <template v-for="(val, key) in data.children">
          <tree-node :data="val"
                     :level="Number(level + 1)"
                     :styleOptions="styleOptions"
                     :settings="settings"
                     :events="events"
                     :asyncLoad="asyncLoad"
                     ref="child"></tree-node>
        </template>
        <!-- 加载中 -->
        <async-loading v-if="isLoading" :styleOptions="styleOptions" :settings="settings"
                       :level="Number(level + 1)"></async-loading>
      </div>
    </transition>
  </div>
</template>
<style scoped>

  /* 列表的根结点的样式（可能有多个根结点）（含他的子节点部分） */
  .root {
    min-width: 100%;
  }

  /* 第一层根结点（只有根节点）（不含其它子节点） */
  .topNode {
    width: 100%;
    font-size: 20px;
    background-color: red;
    position: relative;
    white-space: nowrap;
    overflow: hidden;
  }

  .content {
    position: relative;
    display: inline-block;
    overflow: hidden;
    height: 20px;
    line-height: 20px;
    cursor: pointer;
  }

  .btn-box {
    display: inline-block;
    float: left;
    position: relative;
  }

  .btn-span {
    display: inline-block;
    vertical-align: top;
  }

  .openbtn-nochildren {
    cursor: default;
  }

  .text-box {
    height: 20px;
    line-height: 20px;
    display: inline-block;
    /*margin-left: -3px; !*处理空格带来的空白*!*/
    overflow: hidden;
    position: relative;
  }

  .underline {
    position: absolute;
    left: 0;
    bottom: 1px;
    height: 1px;
    z-index: 1;
    width: 0;
    background-color: #fff;
    width: 0%;
  }

  /* duang的特效 */
  .text-box:hover .underline {
    width: 100%;
    transition: width 1s ease;
  }

  .text {
    display: inline-block;
  }

  .text-is-ellipsis {
    text-overflow: ellipsis;
    overflow: hidden;
  }

</style>
<script>
  import treeNode from './treeNode.vue'
  import asyncLoading from './asyncLoading.vue'
  import {
    DOMTransitionWhenBeforeEnter,
    DOMTransitionWhenEnter,
    DOMTransitionWhenLeave,
    whenMouseOver,
    whenMouseOut,
    setTextSpanWidth,
    getNodeData,
    getSelectedNodeData
  } from './public'

  export default {
    props: {
      data: {
        type: Object
      },
      styleOptions: {
        type: Object,
        default () {
          return {}
        }
      },
      settings: {
        type: Object,
        default () {
          return {}
        }
      },
      events: {
        type: Object,
        default () {
          return {}
        }
      },
      asyncLoad: {
        type: Object,
        default () {
          return {}
        }
      }
    },
    created () {
      if (!this.data.checked) {
        this.data.checked = 0
      }
      // 将当前变量的选中状态，设置给checked
      this.data.checked = Number(this.data.checked)
      this.checkedStatus = this.data.checked
      this.$watch('checkedStatus', function (newValue, oldValue) {
        this.data.checked = newValue
      })
    },
    mounted () {
      this.setTextSpanWidth()
    },
    data () {
      return {
        level: 0,
        isMouseover: false, // 当前鼠标是否移动上去了
        textSpan: {},
        isOpened: true,
        checkedStatus: 0,
        isLoading: false
      }
    },
    methods: {
      // 子节点显示或者隐藏
      hideOrShow () {
        if (this.data.children) {
          this.isOpened = !this.isOpened
        }
        if (this.isOpened && this.asyncLoad.beforeLoad(this.getData(), this.getChildren(), this)) {
          this.asyncLoadData()
        }
      },
      // 选中或者取消选中，会影响子节点和父节点
      changeSelectStatus () {
        // 未选中->选中
        // 半选->选中
        // 选中->未选中
        this.checkedStatus = this.checkedStatus === 2 ? 0 : 2

        // 根节点无需触发该方法
        // this.$parent.whenChildNodeCheckedStatusChanged()
        if (this.$refs.child) {
          this.$refs.child.forEach(child => {
            child.whenParentNodeCheckStatusChanged(this.checkedStatus)
          })
        }
      },
      // 当子节点选中状态变化时，触发本方法
      whenChildNodeCheckedStatusChanged () {
        let count = {
          0: 0,
          1: 0,
          2: 0
        }
        this.$refs.child.forEach(child => {
          count[child.getCheckboxStatus()]++
        })
        // 如果半选的数目大于0，那么就是即有选中也有未选中，当前节点设为半选
        // 剩余情况为只有选中和未选中
        if (count[1] > 0) {
          this.checkedStatus = 1
        } else if (count[0] > 0 && count[2] > 0) {
          // 两种都有的话，就是半选了
          this.checkedStatus = 1
        } else if (count[0] === 0) {
          // 未选中为0说明全选，不然就是未选
          this.checkedStatus = 2
        } else {
          this.checkedStatus = 0
        }
      },
      // 当父节点选中状态变化时，触发本方法
      whenParentNodeCheckStatusChanged (parentNodeIsChecked) {
        // 如果当前状态和父节点传递过来的状态相同，说明无需改变
        if (this.checkedStatus === parentNodeIsChecked) {
          return
        }
        // 设置当前子节点状态为父节点状态，并向下递归传递
        this.checkedStatus = parentNodeIsChecked
        if (this.$refs.child) {
          this.$refs.child.forEach(child => {
            child.whenParentNodeCheckStatusChanged(parentNodeIsChecked)
          })
        }
      },
      // 返回当前子节点的状态
      getCheckboxStatus () {
        return this.checkedStatus
      },
      // 过渡动画，过渡过程中点击无效
      beforeEnter (el) {
        DOMTransitionWhenBeforeEnter(el, this.settings)
      },
      // 进入时执行函数
      enter (el, done) {
        // 这个只执行一次
        DOMTransitionWhenEnter(el, done, this.settings)
      },
      // 退出时执行函数
      leave (el, done) {
        DOMTransitionWhenLeave(el, done, this.settings)
      },
      // 鼠标移动到结点上时，假如结点显示内容超出范围，则自动左移一段距离
      mouseoverTransition () {
        whenMouseOver.apply(this)
      },
      mouseoutTransition () {
        whenMouseOut.apply(this)
      },
      // 单击事件
      clickEvent () {
        this.events.click(this.getData(), this, this.data.children, undefined)
      },
      // 双击事件
      debouleClickEvent () {
        return this.events.dblclick(this.getData(), this, this.data.children, undefined) ? undefined : this.hideOrShow()
      },
      // 鼠标移动上去后的事件
      mouseoverEvent () {
        this.events.mouseover(this.getData(), this, this.data.children, undefined)
      },
      // 鼠标移动走的事件
      mouseoutEvent () {
        this.events.mouseout(this.getData(), this, this.data.children, undefined)
      },
      // 设置文本显示区域的宽度
      setTextSpanWidth () {
        setTextSpanWidth.call(this)
      },
      // 如果改变了根节点的宽度，或者其他需要重绘的情况，需要手动调用这个方法
      resize () {
        // 重绘时需要被触发的函数
        this.setTextSpanWidth()
        if (!this.$refs.child || this.$refs.child.length === 0) {
          return
        }
        this.$refs.child.forEach(child => {
          child.resize()
        })
      },
      // 获取选中的节点的数据
      getSelectedNode () {
        return getSelectedNodeData.call(this)
      },
      // 获取当前节点的数据
      getData () {
        return getNodeData.call(this, this.data, true)
      },
      // 获取根节点组件
      getRootElement () {
        return this
      },
      // 获取children
      getChildren () {
        let arr = []
        if (this.data.children) {
          this.data.children.forEach(item => {
            arr.push(item)
          })
          return arr
        }
        return undefined
      },
      // 获取子组件，有children属性不一定有子组件（因为可能未渲染）
      // 三种返回情况，无返回undefined，有子组件返回子组件，其他情况返回空数组
      getChildrenElement () {
        // 没有children返回undefined
        if (!this.data.children) {
          return undefined
        }
        // 如果打开了，那么返回child
        return this.$refs.child ? this.$refs.child : []
      },
      // 异步加载数据到子组件
      asyncLoadData () {
        new Promise((resolve, reject) => {
          this.isLoading = true
          this.asyncLoad.load(this, resolve, reject)
        }).then(data => {
          // 如果不是数组，那么先设置为数组
          if (Object.prototype.toString.call(this.data.children) !== '[object Array]') {
            this.data.children = []
          }
          if (this.asyncLoad.insert(data, this.data.children)) {
            this.isOpened = !this.isOpened
          }
          this.isLoading = false
        }).catch(err => {
          this.asyncLoad.errorCatch(err)
          this.isLoading = false
        })
      }
    },
    computed: {
      defaultRootStyle () {
        let style = {}
        // 如果overflow隐藏，那么添加overflow
        if (this.settings.isOverflowHidden.enabled) {

        }
        return style
      },
      rootStyle () {
        if (this.styleOptions.listStyle) {
          return Object.assign(this.styleOptions.rootStyle, this.defaultRootStyle)
        } else {
          return this.rootStyle
        }
      },
      topNodeStyle () {
        let style = {}
        style['height'] = this.styleOptions.topNodeStyle.height ? this.styleOptions.topNodeStyle.height : '40px'
        style['line-height'] = this.styleOptions.topNodeStyle.lineHeight ? this.styleOptions.topNodeStyle.lineHeight : '40px'
        style['padding-left'] = this.settings.backSpace.additionalBackSpaceForRootNode + 'px'
        if (this.styleOptions.topNodeStyle) {
          return Object.assign({}, this.styleOptions.topNodeStyle, style)
        } else {
          return style
        }
      },
      contentStyle () {
        let style = {}
        style['height'] = this.styleOptions.topNodeStyle.height ? this.styleOptions.topNodeStyle.height : '40px'
        style['line-height'] = this.styleOptions.topNodeStyle.lineHeight ? this.styleOptions.topNodeStyle.lineHeight : '40px'
        return style
      },
      btnBoxStyle () {
        let style = {}
        style['height'] = this.styleOptions.topNodeStyle.height ? this.styleOptions.topNodeStyle.height : '40px'
        style['line-height'] = this.styleOptions.topNodeStyle.lineHeight ? this.styleOptions.topNodeStyle.lineHeight : '40px'
        return style
      },
      textBoxStyle () {
        let style = {}
        style['height'] = this.styleOptions.topNodeStyle.height ? this.styleOptions.topNodeStyle.height : '40px'
        style['line-height'] = this.styleOptions.topNodeStyle.lineHeight ? this.styleOptions.topNodeStyle.lineHeight : '40px'
        return style
      },
      textStyle () {
        let style = {}
        style['height'] = this.styleOptions.topNodeStyle.height ? this.styleOptions.topNodeStyle.height : '40px'
        style['line-height'] = this.styleOptions.topNodeStyle.lineHeight ? this.styleOptions.topNodeStyle.lineHeight : '40px'
        return Object.assign({}, this.textSpan, style)
      },
      textClass () {
        return {
          'isMouseover': this.isMouseover,
          'text-is-ellipsis': this.settings.isOverflowHidden.enabled && this.settings.isOverflowHidden.isEllipsis
        }
      },
      dropBtnStyle () {
        return Object.assign({}, this.btnBoxStyle, {opacity: (this.data.children ? 1 : 0)})
      },
      checkBoxStyle () {
        return Object.assign({}, this.btnBoxStyle)
      }
    },
    components: {
      treeNode,
      asyncLoading
    }
  }
</script>
