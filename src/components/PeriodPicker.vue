<template>
  <div class="period-picker">
    <button
      v-for="({ v1, v2 }, index) of value"
      :key="`period-picker-${ index }`"></button>
    <button class="add-date"
      v-if="placeholder"
      @click="setPicker">{{ placeholder }}➕</button>
    <div class="picker">
      <div class="mode-select">
        <button 
          :class="{
            'selected-mode': item === mode
          }"
          v-for="(item, index) of modeList"
          :key="`mode-select-${ index }`"
          @click="setMode(item)">{{ item }}</button>
      </div>
      <div class="calender">
        <button class="moving-month"
          @click="setTargetDate(targetDate.add(-1, 'month'))">❰</button>
        <button class="moving-month"
          @click="setTargetDate(targetDate.add(1, 'month'))">❱</button>
        <div class="calender-year">{{ targetDate.year() }}</div>
        <div class="calender-month">{{ targetDate.month() + 1 }}</div>
        <div class="calender-row"
          v-for="(row, index) of targetCalender"
          :key="`calender-row-${ index }`">
          <div class="calender-date"
            v-for="(moment, index) of targetCalender[index]"
            :class="{
              'blur-date': !isTargetMonth(moment),
              'start-date': isStartDate(moment),
              'end-date': isEndDate(moment),
              'included-date': isIncludedDate(moment),
              'pending-date': clickStack % 2,
            }"
            :key="`calender-column-${ index }`"
            @mousedown.prevent="() => {}"
            @click="pickDate(moment.clone())">
            <div class="calender-marker">{{ moment.date() }}</div>
          </div>
        </div>
      </div>
      <div v-show="mode !== 'day'" class="time">
        <div class="time-start">
          <input class="hour-input" type="text" maxlength="2"
            @input="timeInputHandle($event, startDate, startHour, 'hour')"
            @keydown.enter="correctTime($event, startDate, 'hour')"
            @blur="correctTime($event, startDate, 'hour')"
            :value="startHour" />:<input class="min-input" type="text" maxlength="2"
              @input="timeInputHandle($event, startDate, startMinute, 'minute')"
              @keydown.enter="correctTime($event, startDate, 'minute')"
              @blur="correctTime($event, startDate, 'minute')"
              :value="startMinute" />
        </div>
        <div class="time-end">
          <input class="hour-input" type="text" maxlength="2"
            @input="timeInputHandle($event, endDate, endHour, 'hour')"
            @keydown.enter="correctTime($event, endDate, 'hour')"
            @blur="correctTime($event, endDate, 'hour')"
            :value="endHour" />:<input class="min-input" type="text" maxlength="2"
            @input="timeInputHandle($event, endDate, endMinute, 'minute')"
              @keydown.enter="correctTime($event, endDate, 'minute')"
              @blur="correctTime($event, endDate, 'minute')"
              :value="endMinute" />
        </div>
      </div>
      <button>Back</button>
      <button>Confirm</button>
    </div>
  </div>
</template>

<script>
import moment from 'moment';
const comp = (v, m, M) => Math.min(M, Math.max(v, m));
const getDatePiece = moment => {
  const NOW_FORMAT = 'YYYY,MM,DD,HH,mm,ss';
  const [ YYYY, MM, DD, HH, mm ] = moment.format(NOW_FORMAT).split(',');
  return { YYYY, MM, DD, HH, mm };
};
const mkCalendar = target => {
  const calendar = [];
  const startDay = moment(target).clone().startOf('month').startOf('week');
  const endDay = moment(target).clone().endOf('month').endOf('week');
  const date = startDay.clone().subtract(1, 'day');

  while (date.isBefore(endDay, 'day')) calendar.push(Array(7).fill(0).map(() => date.add(1, 'day').clone()));
  return calendar.splice(0);
}

export default {
  props: {
    value: {
      type: Array,
      default: () => [],
    },
    placeholder: {
      type: String,
      default: 'Add Date',
    },
    defaultMode: {
      type: String,
      default: 'day',
    }
  },
  data() {
    return {
      cursor: -1,
      onPicker: false,
      modeList: ['day', 'before', 'after', 'range'],
      mode: 'day',
      targetDate: moment(),
      startDate: null,
      endDate: null,
      clickStack: 0,
    };
  },
  mounted() {
    console.log(this);
    if (this.defaultMode) this.mode = this.defaultMode;
  },
  methods: {
    pickDate(moment) {
      if (!this.isTargetMonth(moment)) this.setTargetDate(moment);

      if (this.mode === 'day') {
        moment.set({
          hour: 0,
          minute: 0,
          second: 0,
          millisecond: 0,
        });
        const nextDate = moment.clone().add(1, 'days');
        this.startDate = moment;
        this.endDate = nextDate;
      }
      if (this.mode === 'before' || this.mode === 'after') {
        this.startDate = this.endDate = moment;
      }
      if (this.mode === 'range') {
        if (this.clickStack % 2 === 0) {
          this.startDate = moment;
          this.endDate = null;
        }
        if (this.clickStack % 2 === 1) {
          this.endDate = moment;
          this.correntRange(); // 시간입력시 체크
        }
        this.clickStack++;
      }
    },
    setPicker() {
      const now = moment();
      this.pickDate(now);
    },
    setMode(name) {
      this.mode = name;
      this.clickStack = 0;
    },
    setTargetDate(moment) {
      this.targetDate = moment.clone();
    },
    isTargetMonth(moment) {
      return this.targetDate.month() === moment.month();
    },
    isStartDate(moment) {
      if (!this.startDate) return;
      return this.startDate.format('YYYYMMDD') === moment.format('YYYYMMDD');
    },
    isEndDate(moment) {
      if (!this.endDate) return;
      if (this.mode !== 'range') return;
      else return this.endDate.format('YYYYMMDD') === moment.format('YYYYMMDD');
    },
    isIncludedDate(moment) {
      if (!this.startDate) return;
      if (this.mode === 'day') return;
      if (this.mode === 'before') {
        return moment.valueOf() < this.startDate.valueOf();
      }
      if (this.mode === 'after') {
        return moment.valueOf() > this.startDate.valueOf();
      }
      if (!this.endDate) return;
      if (this.mode === 'range') {
        return this.startDate.valueOf() < moment.valueOf() && moment.valueOf() < this.endDate.valueOf();
      }
    },
    correntRange() {
      if (this.endDate.valueOf() < this.startDate.valueOf()) {
        [this.startDate, this.endDate] = [this.endDate, this.startDate];
      }
    },
    timeInputHandle(e, targetMoment, tempValue, targetType) {
      const { target } = e;
      const { value } = target;
      if (/^[0-9]*$/.test(value)) {
        if (targetType === 'hour' && 0 <= Number(value) && Number(value) < 24) {
          targetMoment.set({ [targetType]: value });
        }
        if (targetType === 'minute' && 0 <= Number(value) && Number(value) < 60) {
          targetMoment.set({ [targetType]: value });
        }
      } else {
        targetMoment.set({ [targetType]: tempValue });
      }
    },
    correctTime(e, targetMoment, targetType) {
      const { target } = e;
      const { value } = target;  
      if (targetType === 'hour') {
        const computedValue = comp(Number(value), 0, 23);
        targetMoment.set({ [targetType]: computedValue });
        target.value = targetMoment.format('HH');
      }
      if (targetType === 'minute') {
        const computedValue = comp(Number(value), 0, 59);
        targetMoment.set({ [targetType]: computedValue });
        target.value = targetMoment.format('mm');
      }
    }
  },
  computed: {
    targetCalender() {
      return mkCalendar(this.targetDate);
    },
    startHour() {
      if (this.startDate) return this.startDate.format('HH');
      else return '00';
    },
    startMinute() {
      if (this.startDate) return this.startDate.format('mm');
      else return '00';
    },
    endHour() {
      if (this.endDate) return this.endDate.format('HH');
      else return '00';
    },
    endMinute() {
      if (this.endDate) return this.endDate.format('mm');
      else return '00';
    }
  },
};
</script>

<style lang="scss">
.period-picker {
  display: block;
  box-sizing: border-box;
  padding: 5px;
  border: 1px solid #ccc;
  cursor: text;
  position: relative;

  .add-date {
    position: relative;
    background: sandybrown;
    color: white;
    padding-left: 5px;
    line-height: 22px;
    height: 22px;
    border-radius: 5px;
    overflow: hidden;
    border: 0;
    outline: none;
  }
}
.picker {
  cursor: pointer;
  width: 230px;
  height: 270px;
  box-sizing: border-box;
  border: 1px solid rgb(209, 209, 209);

  .mode-select button {
    cursor: pointer;
    border: 0;
    background: transparent;
    outline: none;

    &.selected-mode {
      color: white;
      background: salmon;
      
    }
  }
  .calender {
    .moving-month {
      background: transparent;
      border: 0;
      outline: none;
    }
    .calender-year, .calender-month {
      display: inline-block;
    }
    .calender-date {
      position: relative;
      display: inline-block;
      width: 30px;
      height: 30px;
      
      .calender-marker:hover {
        color: white;
        background: rgb(223, 223, 223);
      }

      &.start-date .calender-marker, &.end-date .calender-marker {
        color: white;
        background: salmon;
      }
      &.included-date .calender-marker {
        color: white;
        background: salmon;
      }

      &.start-date.pending-date .calender-marker {
        color: white;
        background: lighten(salmon, 20);
      }
      &.blur-date {
        color: #ccc;
      }
      &.blur-date.included-date .calender-marker {
        background: lighten(salmon, 20);
      }
      .calender-marker {
        position: absolute;
        width: 100%;
        height: 100%;
        line-height: 30px;
        box-sizing: border-box;
        border-radius: 10px;
        transition: color 0.2s, background 0.2s;
      }
    }
  }
  .time {
    .time-start, .time-end {
      display: inline-block;
    }
    input {
      width: 2em;
      text-align: center;
    }
  }
}
</style>
