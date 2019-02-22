<template>
  <div id="datePicker" @click.stop>
    <input class="input" type="text" v-model="dateValue" @click="openPanel">
    <!-- 选择面板 -->
    <transition name="fadeDownBig">
      <div class="date-panel" v-show="panelState">
         <!-- 第一行 -->
         <div class="topbar">
          <span class="month">{{changeTmpMonth}}</span>
          <span class="year">{{tmpYear}}</span>
          <div class="angel">
            <span @click="left">&lt;</span>
            <span @click="right">&gt;</span>
          </div>
        </div>
        <!-- 日期面板 -->
        <div class="date-group">
          <span v-for="(item, index) in weekList" :key="index" class="weekday">{{item.label}}</span>
        </div>  
        <ul class="date-list">
          <li v-for="(item, index) in dateList"
              v-text="item.value"
              :class="{preMonth: item.previousMonth, nextMonth: item.nextMonth,
              selected: date === item.value && month === tmpMonth && item.currentMonth, invalid: validateDate(item)}"
              :key="index" 
              @click="selectDate(item)">
          </li>
        </ul> 
      </div>
    </transition>
  </div>
</template>

<script>

export default {
  name: 'datePicker',
  data () {
    return {
      dateValue: "", // 输入框内容
      weeksName:['S','M','T','W','T','F','S'],
      date: new Date().getDate(), // 当前日期
      month: new Date().getMonth(), // 当前日期
      panelState: false, // 初始值，默认panel关闭
      tmpMonth: new Date().getMonth(), // 初始月份
      tmpYear: new Date().getFullYear(), // 初始年份，
      weekList: [
        { label: "S", value: 0 },
        { label: "M", value: 1 },
        { label: "T", value: 2 },
        { label: "W", value: 3 },
        { label: "T", value: 4 },
        { label: "F", value: 5 },
        { label: "S", value: 6 }
      ], // 周
      monthList: [
        { label: "January", value: 0 },
        { label: "February", value: 1 },
        { label: "March", value: 2 },
        { label: "April", value: 3 },
        { label: "May", value: 4 },
        { label: "June", value: 5 },
        { label: "July", value: 6 },
        { label: "August", value: 7 },
        { label: "September", value: 8 },
        { label: "October", value: 9 },
        { label: "November", value: 10 },
        { label: "December", value: 11 }
      ], // 月
      nowValue: 0, // 当前选中日期值
    }
  },
  props: {
    format: {
      type: String,
      default: "yyyy-MM-dd"
    }
  },
  computed: {
    dateList() {
      //获取当月的天数
      let currentMonthLength = new Date(
        this.tmpYear,
        this.tmpMonth + 1,
        0
      ).getDate();
      //先将当月的日期塞入dateList
      let dateList = [];
      for(let i = 0;i<currentMonthLength;i++){
        dateList.push({
          currentMonth: true,
          value: i + 1
        });
      }
      // 获取当月1号的星期是为了确定在1号前需要插多少天
      let startDay = new Date(this.tmpYear, this.tmpMonth, 1).getDay();
      // 确认上个月一共多少天
      let previousMongthLength = new Date(
        this.tmpYear,
        this.tmpMonth,
        0
      ).getDate();
      // 在1号前插入上个月日期
      for (let i = 0, len = startDay; i < len; i++) {
        dateList = [
          { previousMonth: true, value: previousMongthLength - i }
        ].concat(dateList);
      }
      // 补全剩余位置
      for (let i = 0, item = 1; i < 14; i++, item++) {
        dateList.push({ nextMonth: true, value: item });
      }
      return dateList;
    },
    changeTmpMonth() {
      return this.monthList[this.tmpMonth].label;
    },
    yearList() {
      return Array.from({ length: 12 }, (value, index) => this.tmpYear + index);
    }
  },
  mounted() {
    if (this.value) {
      this.dateValue = this.formatDate(new Date(this.value).getTime());
    }
    //点击外部面板消失
    window.addEventListener("click", this.eventListener);
  },
  methods: {
    openPanel() {
      this.panelState = !this.panelState;
    },
    left() {
      if (this.tmpMonth === 0) {
        this.tmpYear--;
        this.tmpMonth = 11;
      }
      else this.tmpMonth--;
    },
    right() {
      if (this.tmpMonth === 11) {
        this.tmpYear++;
        this.tmpMonth = 0;
      }
      else this.tmpMonth++;
    },
    eventListener() {
      this.panelState = false;
    },
    validateDate(item) {
      if (this.nowValue === item.value && item.currentMonth) return true;
    },
    selectDate(item) {
      this.nowValue = item.value;
      if (item.previousMonth) this.tmpMonth--;
      if (item.nextMonth) this.tmpMonth++;
      let selectDay = new Date(this.tmpYear, this.tmpMonth, this.nowValue);
      console.log(selectDay.getTime())
      this.dateValue = this.formatDate(selectDay.getTime());
      this.panelState = !this.panelState;
      this.$emit("input", selectDay);
    },
    formatDate(date, fmt = this.format) {
      // 长度为10的时候末尾补3个0
      if (date === null || date === "null") {
        return "--";
      }
      date = new Date(Number(date));
      var o = {
        "M+": date.getMonth() + 1, // 月份
        "d+": date.getDate(), // 日
        "h+": date.getHours(), // 小时
        "m+": date.getMinutes(), // 分
        "s+": date.getSeconds(), // 秒
        "q+": Math.floor((date.getMonth() + 3) / 3), // 季度
        S: date.getMilliseconds() // 毫秒
      };
      if (/(y+)/.test(fmt))
        fmt = fmt.replace(
          RegExp.$1,
          (date.getFullYear() + "").substr(4 - RegExp.$1.length)
        );
      for (var k in o) {
        if (new RegExp("(" + k + ")").test(fmt))
          fmt = fmt.replace(
            RegExp.$1,
            RegExp.$1.length === 1
              ? o[k]
              : ("00" + o[k]).substr(("" + o[k]).length)
          );
      }
      return fmt;
    }
  }
}
</script>

<style lang="scss">
.date-picker {
  width: 210px;
  text-align: center;
}
.input {
  display: inline-block;
  box-sizing: border-box;
  width: 100%;
  height: 32px;
  line-height: 1.5;
  padding: 4px 7px;
  font-size: 12px;
  border: 1px solid #dcdee2;
  border-radius: 4px;
  color: #515a6e;
  background-color: #fff;
  background-image: none;
  position: relative;
  cursor: text;
  transition: border 0.2s ease-in-out, background 0.2s ease-in-out,
    box-shadow 0.2s ease-in-out;
  margin-bottom: 6px;
}

#datePicker {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  width: 210px;
}

.date-panel{
    -webkit-box-shadow: 0 0 8px #ccc;
    background: #fff;
    box-shadow: 0 0 8px #ccc;
    width: 210px;
    overflow: hidden;
}
.topbar {
  padding-top: 8px;
  position: relative;
  overflow: hidden;
  width: 210px;
  .month{
    margin-left: 12px;
    float: left;
  }
  .year{
    margin-left: 12px;
    float: left;
  }
  .angel{
    margin-right: 12px;
    float: right;
  }
}
.weekday {
  display: inline-block;
  font-size: 13px;
  width: 30px;
  color: #9ca5b6;
  text-align: center;
}
.date-list {
  width: 210px;
  text-align: left;
  height: 180px;
  overflow: hidden;
  margin-top: 4px;
}
.date-list li {
  font-size: 13px;
  display: inline-block;
  width: 28px;
  height: 28px;
  line-height: 30px;
  text-align: center;
  cursor: pointer;
  color: #000;
  border: 1px solid #fff;
  border-radius: 50%;
  margin: 0;
}

.date-list .preMonth,
.date-list .nextMonth {
  color: #c5c8ce;
}
.date-list .selected {
  border: 1px solid #2d8cf0;
}
.date-list .invalid {
  background: #2d8cf0;
  color: #fff;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}


a {
  color: #42b983;
}
</style>
