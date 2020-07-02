<template>
  <div class="main">
    <div id="clock">
      <div class="dial" @mousemove="pointerRotate" @mouseup="stopChangePointer">
        <ul id="markList"></ul>
        <span
          class="miao"
          v-bind:style="{transform: 'translate(-50%, 0) rotate('+miaoDeg+'deg)'}"
          @mousedown="mouseDown(1)"
        ></span>
        <span
          class="fen"
          v-bind:style="{transform: 'translate(-50%, 0) rotate('+fenDeg+'deg)'}"
          @mousedown="mouseDown(2)"
        ></span>
        <span
          class="shi"
          v-bind:style="{transform: 'translate(-50%, 0) rotate('+shiDeg+'deg)'}"
          @mousedown="mouseDown(3)"
        ></span>
      </div>
      <div class="time">
        <div class="digital-time">
          <input
            type="button"
            value="+"
            class="digital-time-item"
            @click="changeTimeByButton(3,1,1)"
          />
          <span class="digital-time-item">{{ showHour }}</span>
          <input
            type="button"
            value="-"
            class="digital-time-item"
            @click="changeTimeByButton(3,1,0)"
          />
        </div>
        <div class="digital-time">
          <span>:</span>
        </div>
        <div class="digital-time">
          <input
            type="button"
            value="+"
            class="digital-time-item"
            @click="changeTimeByButton(2,1,1)"
          />
          <span class="digital-time-item">{{ showMinute }}</span>
          <input
            type="button"
            value="-"
            class="digital-time-item"
            @click="changeTimeByButton(2,1,0)"
          />
        </div>
        <div class="digital-time">
          <span>:</span>
        </div>
        <div class="digital-time">
          <input
            type="button"
            value="+"
            class="digital-time-item"
            @click="changeTimeByButton(1,1,1)"
          />
          <span class="digital-time-item">{{ showSecond }}</span>
          <input
            type="button"
            value="-"
            class="digital-time-item"
            @click="changeTimeByButton(1,1,0)"
          />
        </div>
        <!-- <p class="time">{{ time }}</p> -->
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'clock',
  data () {
    return {
      // date: new Date(),
      hour: 0,
      minute: 0,
      second: 0,
      timerID: '',
      isMouseDown: 0,
      haveChanged: 0,
      pointerX: 0,
      pointerY: 0
    }
  },
  computed: {
    // 根据时间值计算对应的角度
    miaoDeg: function () {
      return parseInt(this.second) * 6
    },
    fenDeg: function () {
      return parseInt(this.minute) * 6 + parseInt(this.second) * 0.1
    },
    shiDeg: function () {
      return parseInt(this.minute) * 0.5 + parseInt(this.hour) * 30
    },
    // 补充0之后展示时间
    showSecond: function () {
      return this.zeroPadding(parseInt(this.second), 2)
    },
    showMinute: function () {
      return this.zeroPadding(parseInt(this.minute), 2)
    },
    showHour: function () {
      return this.zeroPadding(parseInt(this.hour), 2)
    }
  },

  watch: {
    // 侦听时间的变化
    minute: function (val) {
      // console.log(val);
      if (val >= 60) {
        this.minute = val - 60
        this.hour++
      } else if (val < 0) {
        this.minute = val + 60
        this.hour--
      }
    },
    second: function (val) {
      // console.log(val);
      if (val >= 60) {
        this.second = val - 60
        this.minute++
      } else if (val < 0) {
        this.second = val + 60
        this.minute--
      }
    },
    hour: function (val) {
      // console.log(val);
      if (val >= 24) {
        this.hour = val - 24
      } else if (val < 0) {
        this.hour = val + 24
      }
    }
  },

  mounted () {
    this.addMarks()
    this.updateTime()
    this.timerID = setInterval(this.updateTime, 1000)
  },
  methods: {
    // 添加钟表刻度
    addMarks () {
      let mList = document.getElementById('markList')
      let marks = ''
      let sCss = ''
      for (let i = 0; i < 60; i++) {
        sCss = 'transform: rotate(' + i * 6 + 'deg);'
        marks += "<li style='" + sCss + "'></li>"
      }
      mList.innerHTML = marks
    },

    // 更新时间
    updateTime () {
      let cd = new Date()
      this.hour = cd.getHours()
      this.minute = cd.getMinutes()
      this.second = cd.getSeconds()
    },

    // 修改时间
    changeTime (p, time, tag) {
      let s = this.second
      let m = this.minute
      let h = this.hour
      // 根据参数判断时间的更改
      if (tag === 1) {
        switch (p) {
          case 1:
            this.second = s + time
            // this.afterChangeTime();
            break
          case 2:
            this.minute = m + time
            // this.afterChangeTime();
            break
          case 3:
            this.hour = h + time
            // this.afterChangeTime();
            break
        }
      }
      if (tag === 0) {
        switch (p) {
          case 1:
            this.second = s - time
            // this.afterChangeTime();
            break
          case 2:
            this.minute = m - time
            // this.afterChangeTime();
            break
          case 3:
            this.hour = h - time
            // this.afterChangeTime();
            break
        }
      }
    },

    // 按钮修改时间
    changeTimeByButton (p, time, tag) {
      if (this.haveChanged === 0) {
        clearInterval(this.timerID)
        this.timerID = setInterval(() => {
          this.changeTime(1, 1, 1)
        }, 1000)
        // console.log("使用按钮改变过时间！")
        this.haveChanged = 1
      }
      this.changeTime(p, time, tag)
      // this.haveChanged = 1;
    },

    // 补充0
    zeroPadding (num, digit) {
      let zero = ''
      for (let i = 0; i < digit; i++) {
        zero += '0'
      }
      return (zero + num).slice(-digit)
    },

    // 获取鼠标在四象限上的位置角度
    getAngle (px, py, mx, my) {
      let x = Math.abs(px - mx)
      let y = Math.abs(py - my)
      let z = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2))
      let cos = y / z
      let radina = Math.acos(cos) //  用反三角函数求弧度
      let angle = Math.floor(180 / (Math.PI / radina)) // 将弧度转换成角度
      if (mx > px && my > py) {
        //  鼠标在第四象限
        angle = 180 - angle
      }
      if (mx === px && my > py) {
        //  鼠标在y轴负方向上
        angle = 180
      }
      if (mx > px && my === py) {
        //  鼠标在x轴正方向上
        angle = 90
      }
      if (mx < px && my > py) {
        //  鼠标在第三象限
        angle = 180 + angle
      }
      if (mx < px && my === py) {
        //  鼠标在x轴负方向
        angle = 270
      }
      if (mx < px && my < py) {
        //  鼠标在第二象限
        angle = 360 - angle
      }
      return angle
    },

    // 获取鼠标转动角度
    getAngle2 (px, py, ox, oy, nx, ny) {
      // 第一条边的平方
      let n1 =
        Math.abs(ox - px) * Math.abs(ox - px) +
        Math.abs(oy - py) * Math.abs(oy - py)
      let n2 =
        Math.abs(ox - px) * Math.abs(ox - px) +
        Math.abs(oy - py) * Math.abs(oy - py)
      let n3 =
        Math.abs(ox - nx) * Math.abs(ox - nx) +
        Math.abs(oy - ny) * Math.abs(oy - ny)
      let cos = (n1 + n2 - n3) / (2 * Math.sqrt(n1) * Math.sqrt(n2))
      let radina = Math.acos(cos) // 用反三角函数求弧度
      let angle = Math.floor(180 / (Math.PI / radina)) // 将弧度转换成角度
      return angle
    },

    /* 鼠标点击指针转动事件
    changePointer(pointer, $event) {
      // const this_ = this;
      if (this.isMouseDown == 1) {
        //只有鼠标按下时才执行
        // 获取旋转中心坐标
        let p = document.querySelector(".dial");
        let x = p.offsetLeft + p.offsetWidth / 2;
        let y = p.offsetTop + p.offsetHeight / 2;
        // 添加
        let ang = this.getAngle(x, y, event.clientX, event.clientY);
        // console.log(ang);
        // console.log(pointer);
        switch (pointer) {
          case 1:
            this.second = Math.floor(ang / 6);
            // console.log(this_.time.second);
            break;
          case 2:
            this.minute = Math.floor(ang / 6);
            break;
          case 3:
            this.hour = Math.floor(ang / 30);
            break;
        }
      } else {
        // 不做任何事
      }
    }, */

    // 鼠标按下事件
    mouseDown (p, $event) {
      // const this_ = this
      this.isMouseDown = p
      clearInterval(this.timerID)
      this.pointerX = event.clientX
      this.pointerY = event.clientY
    },

    //  获取旋转角度并判断正负
    pointerRotate (e) {
      // 获取钟表中心位置
      let p = document.querySelector('.dial')
      let x = p.offsetLeft + p.offsetWidth / 2
      let y = p.offsetTop + p.offsetHeight / 2
      let ang
      // 判断按下
      let md = this.isMouseDown
      if (md !== 0) {
        /*
          1.分离角度，并判断正负
          2.根据角度再修改时间，注意时的角度与分秒不同
        */
        // 旧坐标
        let oldang = this.getAngle(x, y, this.pointerX, this.pointerY)
        // 新坐标
        let newang = this.getAngle(x, y, e.clientX, e.clientY)
        // 判断临界情况
        ang = newang - oldang
        // console.log('角度改变：' + ang)
        if (ang < -350) {
          ang = 360 + ang
        }
        if (ang > 350) {
          ang = ang - 360
        }
        if (ang > 0) {
          if (md === 3) {
            this.changeTime(md, ang / 30, 1)
          } else {
            this.changeTime(md, ang / 6, 1)
          }
          // console.log("增加"+Math.floor(ang / 6)+"秒");
        }
        if (ang < 0) {
          ang = 0 - ang
          if (md === 3) {
            this.changeTime(md, ang / 30, 0)
          } else {
            this.changeTime(md, ang / 6, 0)
          }
          // console.log("减少"+Math.floor(ang / 6)+"秒");
        }
        this.pointerX = e.clientX
        this.pointerY = e.clientY
      }
    },

    // 鼠标松开事件
    stopChangePointer () {
      // window.removeEventListener("mousemove", this.changeAngle(1));
      this.isMouseDown = 0
      this.timerID = setInterval(this.updateTime, 1000)
      this.haveChanged = 0
    }
  }
}
</script>

<style lang='scss'>
ul {
  margin: 0 auto;
  padding: 0;
  list-style: none;
}

#markList {
  li {
    position: absolute;
    width: 2px;
    height: 5px;
    background: #000;
    transform-origin: center 90px;
    left: 98px;
    top: 10px;
  }
  li:nth-of-type(5n + 1) {
    height: 10px;
  }
}

#clock {
  width: 100%;
  color: #ffffff;
  text-align: center;
  color: #000000;

  .dial {
    position: relative;
    width: 200px;
    height: 200px;
    margin: 10px auto;
    border: solid 4px blue;
    border-radius: 50%;

    .miao {
      position: absolute;
      left: 50%;
      top: 10%;
      width: 2%;
      height: 40%;
      /* border: solid 2px #000000; */
      background-color: red;
      transform-origin: bottom;
      z-index: 3;
    }

    .fen {
      position: absolute;
      left: 50%;
      top: 20%;
      width: 2%;
      height: 30%;
      background-color: #000;
      transform-origin: bottom;
      z-index: 2;
    }

    .shi {
      position: absolute;
      left: 50%;
      top: 30%;
      width: 3%;
      height: 20%;
      background-color: #000;
      transform-origin: bottom;
      z-index: 1;
    }
  }

  .time {
    width: 20%;
    min-width: 150px;
    margin: 20px auto;
    text-align: center;

    .digital-time {
      display: inline-block;
      vertical-align: middle;
      width: 20%;
      span {
        letter-spacing: 0.05em;
        font-size: 150%;
        padding: 5px 0;
      }
      .digital-time-item {
        display: block;
        width: 100%;
        height: 33.3%;
      }
    }
  }
}
</style>
