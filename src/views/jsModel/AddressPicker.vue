<template>
  <div ref="addressPicker" class="addressPicker">
    <div class="pick-type-button">
      <div ref="cityPicker" style="position:relative;">
        <div class="initial-address">{{chooseList[0].name}}</div>
        <input type="text" id="city-picker" class="address-picker elliptical" :style="{'opacity':chooseList[0].name.length === 0?'1':'0'}" @click="showCityPicker">
      </div>
      <div @click="showPickList('workType')">{{ workType }}</div>
      <div @click="showPickList('issueDate')">{{ issueDate }}</div>
    </div>
    <div class="picker">
      <div class="work-type">
        <transition name="work-type">
          <ul class="pick-list" ref="workType" v-if="currentPicker === 'workType'">
            <li @click="workTypePickHandle('不限')">
              不限<span v-show="workType == '所有工作类型'">√</span>
            </li>
            <li @click="workTypePickHandle('实习')">
              实习<span v-show="workType == '实习'">√</span>
            </li>
            <li @click="workTypePickHandle('兼职')">
              兼职<span v-show="workType == '兼职'">√</span>
            </li>
            <li @click="workTypePickHandle('全职')">
              全职<span v-show="workType == '全职'">√</span>
            </li>
          </ul>
        </transition>
      </div>
      <div class="issue-date">
        <transition name="issue-date">
          <ul class="pick-list" ref="issueDate" v-if="currentPicker === 'issueDate'">
            <li @click="issueDatePickHandle('不限')">
              不限<span v-show="issueDate == '所有发布时间'">√</span>
            </li>
            <li @click="issueDatePickHandle('3天')">
              3天<span v-show="issueDate == '3天'">√</span>
            </li>
            <li @click="issueDatePickHandle('7天')">
              7天<span v-show="issueDate == '7天'">√</span>
            </li>
            <li @click="issueDatePickHandle('14天')">
              14天<span v-show="issueDate == '14天'">√</span>
            </li>
            <li @click="issueDatePickHandle('30天')">
              30天<span v-show="issueDate == '30天'">√</span>
            </li>
          </ul>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'address-picker',
  data () {
    return {
      address: '',
      hasTimer: false,
      currentPicker: '',
      workType: '所有工作类型',
      issueDate: '所有发布时间',
      chooseList: [
        { id: 1, name: '所有工作地' },
        { id: 2, name: '所有工作类型' },
        { id: 3, name: '所有发布时间' }
      ]
    }
  },
  created () {
    this.$nextTick(() => {
      // eslint-disable-next-line
      $('#city-picker').cityPicker({
        title: '请选择地址',
        showDistrict: false,
        onChange: (picker, values, displayValues) => { // 选择后触发的事件
          this.address = displayValues
        }
      })
    })
  },
  methods: {
    closePickerHandle (e) {
      if (this.$refs.workType) {
        this.$refs.workType.style.animationName = 'wrapper-gradient-hide'
      } else if (this.$refs.issueDate) {
        this.$refs.issueDate.style.animationName = 'wrapper-gradient-hide'
      }
      this.currentPicker = ''
    },
    showPickList (pickType) {
      if (this.currentPicker === '') {
        window.addEventListener('click', this.closePickerHandle, true)
      }
      this.currentPicker = pickType
    },
    workTypePickHandle (value) {
      this.currentPicker = ''
      this.workType = value === '不限' ? '所有工作类型' : value
    },
    issueDatePickHandle (value) {
      this.currentPicker = ''
      this.issueDate = value === '不限' ? '所有发布时间' : value
    },
    showCityPicker () {
      this.chooseList[0].name = ''
      this.hasTimer = false
      setTimeout(() => {
        this.addUnlimitedAddress()
      }, 100)
    },
    addUnlimitedAddress () {
      let unlimited = `<div class="unlimited">不限</div>`
      // eslint-disable-next-line
      $('.weui-picker-container .toolbar-inner').append(unlimited)
      // eslint-disable-next-line
      $('.weui-picker-container .toolbar-inner .unlimited').on('click', () => {
        this.address = null
        this.chooseList[0].name = '所有工作地'

        // eslint-disable-next-line
        $('.weui-picker-container .toolbar-inner .close-picker').trigger('click')
      })
    }
  },
  watch: {
    // 在（所有工作地）时，就会触发
    // eslint-disable-next-line
    address(newValue,oldValue) {
      // 如果已经有定时器了，就返回
      if (this.hasTimer) return
      // 即将开启一个定时器，设置定时器为hasTimer为true
      this.hasTimer = true
      // 开启定时器，每0.1秒检测picker是否关闭
      let timer = setInterval(() => {
        // eslint-disable-next-line
        let havePicker = $('.weui-picker-container').length
        // 如果picker已经关闭，则清除定时器
        if (havePicker === 0) {
          clearInterval(timer)
          // 在这里发送请求
          console.log('在这里发送请求')
        }
      }, 100)
    },
    currentPicker (newValue, oldValue) {
      if (newValue === '') {
        console.log('移除')
        window.removeEventListener('click', this.closePickerHandle, true)
      }
    }
  },
  mounted () {
    this.$Tools.getElementTouchMovelength(this.$refs.addressPicker, (slipeDirection) => {
      if (slipeDirection.left) {
        this.$router.back()
      }
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
// reset css
/deep/.picker-items-col {
  touch-action: none;
}
ul,
li {
  list-style: none;
}

// less css
.addressPicker {
  height: calc(100vh - 45px);
}
.pick-type-button {
  height: 48px;
  background: rgba(255, 255, 255, 1);
  border-top: 1px solid #dddddd;
  border-bottom: 1px solid #dddddd;
  box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.04);
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.work-type,
.issue-date {
  position: absolute;
  overflow: hidden;
  width: 100%;
  .pick-list {
    animation-fill-mode: forwards;
  }
}
.picker {
  position: relative;
  .pick-list {
    width: 100%;
    font-size: 14px;
    line-height: 40px;
    text-align: left;
    box-sizing: border-box;
    border-radius: 0 0 3px 3px;
    box-sizing: border-box;
    background-color: #fff;
    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0 10px;
      border-bottom: 1px solid #dddddd;
    }
  }
}
.address-picker {
  width: 96px;
  height: 25px;
  border: none;
  outline: none;
  font-size: 16px;
  position: relative;
  background-color: transparent;
  z-index: 9;
}

.initial-address {
  position: absolute;
}

.work-type-enter-active {
  animation: wrapper-gradient .5s ease;
}
.work-type-leave-active {
  animation: wrapper-gradient-hid .5s ease;
}
.issue-date-enter-active {
  animation: wrapper-gradient .5s ease;
}
.issue-date-leave-active {
  animation: wrapper-gradient-hide .1s ease;
}
</style>
