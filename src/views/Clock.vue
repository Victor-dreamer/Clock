<template>
  <div class="main">
    <div id="clock">
      <div class="dial" @mousemove="changePointer(isMouseDown)" @mouseup="stopChangePointer">
        <ul id="markList"></ul>
        <span
          class="miao"
          v-bind:style="{transform: 'translate(-50%, 0) rotate('+miaoDeg+'deg)'}"
          @mousedown="mouseDown(1)"
        ></span>
        <span class="fen" v-bind:style="{transform: 'translate(-50%, 0) rotate('+fenDeg+'deg)'}"  @mousedown="mouseDown(2)"></span>
        <span class="shi" v-bind:style="{transform: 'translate(-50%, 0) rotate('+shiDeg+'deg)'}"  @mousedown="mouseDown(3)"></span>
      </div>
      <div class="time">
        <div class="digital-time">
          <input type="button" value="+" class="digital-time-item" @click="changeTime(3,1)" />
          <span class="digital-time-item">{{ showHour }}</span>
          <input type="button" value="-" class="digital-time-item" @click="changeTime(3,0)" />
        </div>
        <div class="digital-time">
          <span>:</span>
        </div>
        <div class="digital-time">
          <input type="button" value="+" class="digital-time-item" @click="changeTime(2,1)" />
          <span class="digital-time-item">{{ showMinute }}</span>
          <input type="button" value="-" class="digital-time-item" @click="changeTime(2,0)" />
        </div>
        <div class="digital-time">
          <span>:</span>
        </div>
        <div class="digital-time">
          <input type="button" value="+" class="digital-time-item" @click="changeTime(1,1)" />
          <span class="digital-time-item">{{ showSecond }}</span>
          <input type="button" value="-" class="digital-time-item" @click="changeTime(1,0)" />
        </div>
        <!-- <p class="time">{{ time }}</p> -->
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "clock",
  data() {
    return {
      // date: new Date(),
      time: {
        hour: 0,
        minute: 0,
        second: 0
      },
      timerID: "",
      isMouseDown: 0,
      haveChanged: 0
    };
  },
  computed: {
    // 根据时间值计算对应的角度
    miaoDeg: function() {
      return this.time.second * 6;
    },
    fenDeg: function() {
      return this.time.minute * 6 + this.time.second * 0.1;
    },
    shiDeg: function() {
      return this.time.minute * 0.5 + this.time.hour * 30;
    },
    showSecond: function(){
      return this.zeroPadding(this.time.second,2);
    },
    showMinute: function(){
      return this.zeroPadding(this.time.minute,2);
    },
    showHour: function(){
      return this.zeroPadding(this.time.hour,2);
    }
  },
  mounted() {
    this.addMarks();
    this.updateTime();
    this.timerID = setInterval(this.updateTime, 1000);
  },
  methods: {
    // 添加钟表刻度
    addMarks() {
      let mList = document.getElementById("markList");
      let marks = "";
      let sCss = "";
      for (let i = 0; i < 60; i++) {
        sCss = "transform: rotate(" + i * 6 + "deg);";
        marks += "<li style='" + sCss + "'></li>";
      }
      mList.innerHTML = marks;
    },

    // 更新时间
    updateTime() {
      let cd = new Date();
      this.time = {
        hour: cd.getHours(),
        minute: cd.getMinutes(),
        second: cd.getSeconds()
      };
    },

    // 时间改变后对各个时间进行进制调整
    afterChangeTime() {
      let s = parseInt(this.time.second);
      let m = parseInt(this.time.minute);
      let h = parseInt(this.time.hour);
      if (s >= 60) {
        s = 0;
        m++;
      } else if (s < 0) {
        s = s + 60;
        m--;
      }
      if (m >= 60) {
        m = 0;
        h++;
      } else if (m < 0) {
        m = m + 60;
        h--;
      }
      if (h >= 24) {
        h = 0;
      } else if (h < 0) {
        h = h + 24;
      }
      this.time = {
        hour: h,
        minute: m,
        second: s,
      };
    },

    // 修改时间
    changeTime(time, tag) {
      const this_ = this;
      let s = parseInt(this_.time.second);
      let m = parseInt(this_.time.minute);
      let h = parseInt(this_.time.hour);
      this_.isChange();
      if (tag == 1) {
        switch (time) {
          case 1:
            this_.time.second = s + 1;
            this_.afterChangeTime();
            break;
          case 2:
            this_.time.minute = m + 1;
            this_.afterChangeTime();
            break;
          case 3:
            this_.time.hour = h + 1;
            this_.afterChangeTime();
            break;
        }
      }
      if (tag == 0) {
        switch (time) {
          case 1:
            this_.time.second = s - 1;
            this_.afterChangeTime();
            break;
          case 2:
            this_.time.minute = m - 1;
            this_.afterChangeTime();
            break;
          case 3:
            this_.time.hour = h - 1;
            this_.afterChangeTime();
            break;
        }
      }
    },

    // 补充0
    zeroPadding(num, digit) {
      let zero = "";
      for (let i = 0; i < digit; i++) {
        zero += "0";
      }
      return (zero + num).slice(-digit);
    },

    //按钮点击事件
    isChange() {
      const this_ = this;
      if (this_.haveChanged == 0) {
        clearInterval(this_.timerID);
        this_.timerID = setInterval(function() {
          this_.changeTime(1, 1);
        }, 1000);
        this_.haveChanged = 1;
      }
    },

    // 获取鼠标转动角度
    getAngle(px, py, mx, my) {
      var x = Math.abs(px - mx);
      var y = Math.abs(py - my);
      var z = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2));
      var cos = y / z;
      var radina = Math.acos(cos); //用反三角函数求弧度
      var angle = Math.floor(180 / (Math.PI / radina)); //将弧度转换成角度
      if (mx > px && my > py) {
        //鼠标在第四象限
        angle = 180 - angle;
      }
      if (mx == px && my > py) {
        //鼠标在y轴负方向上
        angle = 180;
      }
      if (mx > px && my == py) {
        //鼠标在x轴正方向上
        angle = 90;
      }
      if (mx < px && my > py) {
        //鼠标在第三象限
        angle = 180 + angle;
      }
      if (mx < px && my == py) {
        //鼠标在x轴负方向
        angle = 270;
      }
      if (mx < px && my < py) {
        //鼠标在第二象限
        angle = 360 - angle;
      }
      return angle;
    },

    // 指针转动事件
    changePointer(pointer, $event) {
      const this_ = this;
      if (this_.isMouseDown == 1) {
        //只有鼠标按下时才执行
        // 获取旋转中心坐标
        let p = document.querySelector(".dial");
        let x = p.offsetLeft + p.offsetWidth / 2;
        let y = p.offsetTop + p.offsetHeight / 2;
        // 添加
        var ang = this_.getAngle(x, y, event.clientX, event.clientY);
        // console.log(ang);
        console.log(pointer);
        switch (pointer) {
          case 1:
            this_.time.second = Math.floor(ang / 6);
            console.log(this_.time.second);
            break;
          case 2:
            this_.time.minute = Math.floor(ang / 6);
            break;
          case 3:
            this_.time.hour = Math.floor(ang / 30);
            break;
        }
      } else {
        // 不做任何事
      }
    },
    // 鼠标按下事件
    mouseDown(p) {
      const this_ = this
      this_.isMouseDown = p;
      clearInterval(this_.timerID);
    },

    //获取改变的角度并转化为时间
    changeAngle($event, pointer, x, y) {
      const this_ = this;
      let ang = this_.getAngle(x, y, event.clientX, event.clientY);
      switch (pointer) {
        case 1:
          this_.time.second = this_.zeroPadding(Math.floor(ang / 6), 2);
          break;
        case 2:
          this_.time.minute = this_.zeroPadding(Math.floor(ang / 6), 2);
          break;
        case 3:
          this_.time.hour = this_.zeroPadding(Math.floor(ang / 30), 2);
          break;
      }
    },

    stopChangePointer() {
      // window.removeEventListener("mousemove", this.changeAngle(1));
      this.isMouseDown = 0;
      this.timerID = setInterval(this.updateTime, 1000);
      this.haveChanged = 0;
    }
  }
};
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
