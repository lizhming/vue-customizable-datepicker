<template>
  <div
    class="calendar"
    :class="{'isFillRangeEnabled': fillRange, 'isAllwaysOpened': opened, 'isOpened': isCalendarOpened}"
    v-click-outside="closeCalendar"
  >
    <div class="calendar__Input" @click="toggleCalendar" v-if="!opened">
      <div v-if="range && rangeDates && rangeDates.from && rangeDates.to">
        <slot name="value" v-bind="{from: rangeDates.from, to: rangeDates.to}">
          {{ formatDate(rangeDates.from) }}
          -
          {{ formatDate(rangeDates.to) }}
        </slot>
      </div>
      <div v-else-if="selectedDate && pickerValue">
        <slot name="value" v-bind="{date: selectedDate}">
          {{ formatDate(selectedDate) }}
        </slot>
      </div>
      <span v-else v-text="placeholder"/>
    </div>
    <div class="calendar__Items" v-show="isCalendarOpened">
      <div class="calendar__Item" v-for="calendar in calendars" :key="calendar.id" @click.stop>
        <div class="calendar__Header">
          <template v-if="!calendar.isSelectMonthOpened && !calendar.isSelectYearOpened">
            <button
              v-if="arrows"
              @click.prevent.stop="monthArrowClicked(calendar, 'previous')"
              class="calendar__Button calendar__Previous"
            >
              <slot name="arrow-month-previous">
                <span><</span>
              </slot>
            </button>
            <div class="calendar__HeaderDate" @click="openSelectors(calendar)">
              <slot name="header" v-bind="{month: calendar.month, year: calendar.year}">
                <span>{{ calendar.month }} - {{ calendar.year }}</span>
              </slot>
            </div>
            <button
              v-if="arrows"
              @click.prevent.stop="monthArrowClicked(calendar, 'next')"
              class="calendar__Button calendar__Next"
            >
              <slot name="arrow-month-next">
                <span>></span>
              </slot>
            </button>
          </template>
          <template v-else>
            <div class="calendar__Months" v-if="calendar.isSelectMonthOpened">
              <ul>
                <li
                  v-for="m in 12"
                  v-text="zeroPad ? zeroPadNumber(m) : m"
                  :key="m"
                  :class="{'isSelected': calendar.month === (zeroPad ? zeroPadNumber(m) : m)}"
                  @click.stop="setCalendarMonth(calendar, m)"
                />
              </ul>
            </div>
            <div class="calendar__Years" @click.stop v-else-if="calendar.isSelectYearOpened">
              <button
                class="calendar__Button calendar__Previous"
                @click.prevent.stop="generateYears('previous')"
              >
                <slot name="arrow-year-previous">
                  <span><</span>
                </slot>
              </button>
              <button class="calendar__Button calendar__Next" @click.prevent.stop="generateYears('next')">
                <slot name="arrow-year-next">
                  <span>></span>
                </slot>
              </button>
              <ul>
                <li
                  v-for="y in years"
                  v-text="y"
                  :key="y"
                  :class="{'isSelected': calendar.year === y}"
                  @click.stop="setCalendarYear(calendar, y)"
                />
              </ul>
            </div>
          </template>
        </div>
        <div class="calendar">
          <span class="calendar__DayNames">
            <span v-for="d in 7" v-text="dayNamesMerged[d-1]" :key="d" class="calendar__DayName" />
          </span>
          <span v-for="week in calendar.weeks" :key="week.id" class="calendar__Week">
            <span
              v-for="day in week"
              :key="day.id"
              class="calendar__Day"
              :class="{'isSelected': compareDates(day, selectedDate), 'isToday': compareDates(day, now), 'isInRange': isInRange(day), 'isDisabled': isDisabled(day), 'isDifferentMonth': isDifferentMonth(day, calendar.weeks[0]), 'isFirstInRange' :range &&  compareDates(day, rangeDates.from), 'isLastInRange' : range && compareDates(day, rangeDates.to)}"
              @click="selectDate(day)"
            >
              <template v-if="day !== 0">
                <slot
                  name="day"
                  v-bind="{day: zeroPadDays ? zeroPadNumber(day.getDate()) : day.getDate()}"
                >{{ zeroPadDays ? zeroPadNumber(day.getDate()) : day.getDate() }}</slot>
              </template>
            </span>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'
import vClickOutside from 'v-click-outside'
Vue.use(vClickOutside)

let now = new Date();
let year = now.getFullYear();
let month = now.getMonth() + 1;
now.setHours(0, 0, 0, 0);

const getDatesRange = (n, year, month) =>
  [...Array(n).keys()].map(i => new Date(year, month - 1, i + 1));

const getWeekChunks = (year, month, fillEmpty, firstDayNumber) => {
  let numberOfDays = new Date(year, month, 0).getDate();
  let range = getDatesRange(numberOfDays, year, month);
  let firstDay = new Date(year, month - 1, 1).getDay();
  firstDay = firstDay === 0 ? 7 : firstDay;

  let numberOfPadding = 7 - firstDayNumber + firstDay;
  numberOfPadding = numberOfPadding % 7;

  for (let i = 0; i < numberOfPadding; i++) {
    if (fillEmpty) range.unshift(new Date(year, month - 1, -i));
    else range.unshift(0);
  }

  let chunks = getDatesRange(
    Math.ceil(range.length / 7),
    year,
    month
  ).map((x, i) => range.slice(i * 7, i * 7 + 7));

  if (fillEmpty) {
    let emptyPlaces = chunks[chunks.length - 1].length;
    for (let i = 0; i < 7 - emptyPlaces; i++)
      chunks[chunks.length - 1].push(new Date(year, month, i + 1));
  }

  return chunks;
};

export default {
  name: 'DatePicker',
  props: {
    value: {},
    range: {
      type: Boolean,
      default: false
    },
    exactRangeDates: {
      type: Boolean,
      default: false
    },
    numberOfCalendars: {
      type: Number,
      default: 1
    },
    firstDay: {
      type: Number,
      default: 1,
      validator: value => {
        if(value > 6 || value < 0) throw new Error('Property firstDay should be a Number between 0 and 6.')
        return value <= 6 && value >= 0
      }
    },
    disabledDates: {
      type: Array,
      default: () => {
        return [];
      }
    },
    format: {
      type: String,
      default: "%d.%m.%Y"
    },
    dayNames: {
      type: [Array, Object],
      default: () => {
        return {
          1: "Mo",
          2: "Tu",
          3: "We",
          4: "Th",
          5: "Fr",
          6: "Sa",
          0: "Su"
        };
      },
      validator: value => {
        let hasAllKeys = [...Array(7).keys()].every(i => value[i]);
        if(!hasAllKeys) throw new Error('Property dayNames should be an Object with this properties: 0, 1, 2, 3, 4, 5, 6.')
        return hasAllKeys
      }
    },
    fillEmpty: {
      type: Boolean,
      default: true
    },
    fillRange: {
      type: Boolean,
      default: false
    },
    opened: {
      type: Boolean,
      default: false
    },
    closeOnClick: {
      type: Boolean,
      default: true
    },
    zeroPad: {
      type: Boolean,
      default: true
    },
    zeroPadDays: {
      type: Boolean,
      default: false
    },
    arrows: {
      type: Boolean,
      default: true
    },
    placeholder: {
      type: String,
      default: 'Pick a date'
    },
    disableBefore: {},
    disableAfter: {},
  },

  data() {
    return {
      now,
      year,
      month,
      calendars: [],
      pickerValue: this.value,
      years: [...Array(9).keys()].map(i => i + now.getFullYear()),
      selectedDate:
        this.value && !this.value.from && !Array.isArray(this.value)
          ? this.value
          : undefined,
      formatedDate: undefined,
      clickedDates: [],
      isCalendarOpened: this.opened,
      rangeDates: {
        from:
          !Array.isArray(this.value) &&
          this.value &&
          !this.exactRangeDates &&
          this.value.from
            ? this.value.from
            : undefined,
        to:
          !Array.isArray(this.value) &&
          this.value &&
          !this.exactRangeDates &&
          this.value.to
            ? this.value.to
            : undefined
      }
    };
  },

  computed: {
    dayNamesMerged() {
      return [...Array(7).keys()].map(i => this.dayNames[(this.firstDay + i) % 7])
    }
  },

  created() {
    this.setCalendar(undefined, true);
  },

  methods: {
    getWeekChunks(year, month) {
      return getWeekChunks(year, month, this.fillEmpty, this.firstDay);
    },

    generateYears(direction) {
      let first = this.years[0] - 9 * (direction === "next" ? -1 : 1);
      this.years = [...Array(9).keys()].map(i => i + first);
    },

    monthArrowClicked(calendar, direction) {
      let temp = new Date(this.findFirstDay(calendar.weeks[0]).getTime());
      temp.setMonth(temp.getMonth() + (direction === "next" ? 1 : -1));

      this.setCalendarMonth(calendar, temp.getMonth() + 1, false);
      this.setCalendarYear(calendar, temp.getFullYear(), false);
      this.setCalendar(calendar, false);
      this.$emit('month-change', {month: temp.getMonth() + 1, year: temp.getFullYear()});
    },

    setCalendarMonth(calendar, month, fromPicker = true) {
      calendar.month = this.zeroPad ? this.zeroPadNumber(month) : month;
      if (fromPicker) this.setCalendar(calendar, false, "isSelectMonthOpened");
    },

    setCalendarYear(calendar, year, fromPicker = true) {
      if(calendar.year !== year) this.$emit('year-change', {month: parseInt(calendar.month), year});
      calendar.year = year;
      if (fromPicker) {
        this.setCalendar(calendar, false, "isSelectYearOpened");
        this.years = [...Array(9).keys()].map(i => i + now.getFullYear());
      }
    },

    setCalendar(calendar, isInitialization = false, selector) {
      if (isInitialization) {
        let startDate = undefined;
        if (this.pickerValue) {
          if (this.pickerValue.from)
            startDate = new Date(this.pickerValue.from.getTime());
          else if (Array.isArray(this.pickerValue) && Array.isArray.length > 0)
            startDate = new Date(this.pickerValue[0].getTime());
          else startDate = new Date(this.pickerValue.getTime());
        } else startDate = new Date(this.now.getTime());

        for (let i = 0; i < this.numberOfCalendars; i++) {
          let year = startDate.getFullYear();
          let month = startDate.getMonth() + 1;
          if (this.zeroPad) month = this.zeroPadNumber(month);

          this.calendars.push({
            year,
            month,
            weeks: getWeekChunks(year, month, this.fillEmpty, this.firstDay),
            isSelectMonthOpened: false,
            isSelectYearOpened: false
          });
          startDate.setMonth(startDate.getMonth() + 1);
        }
      } else {
        if (selector === "isSelectMonthOpened")
          calendar.isSelectYearOpened = true;
        if (selector) calendar[selector] = false;
        calendar.weeks = getWeekChunks(
          calendar.year,
          calendar.month,
          this.fillEmpty,
          this.firstDay
        );
      }
    },

    selectDate(date) {
      if (date !== 0) {
        this.selectedDate = date;
        if (this.range) {
          if (this.clickedDates.length === 0) {
            this.rangeDates.from = undefined;
            this.rangeDates.to = undefined;
            if (this.exactRangeDates) this.pickerValue = [];
          }
          this.clickedDates.push(date);
          if (this.clickedDates.length === 2) {
            this.clickedDates.sort((a, b) => a - b);
            this.rangeDates.from = this.clickedDates[0];
            this.rangeDates.to = this.clickedDates[1];
            this.clickedDates = [];
            this.selectedDate = undefined;
            let pickerValue = [];
            if (!this.exactRangeDates) {
              pickerValue = {
                from: this.rangeDates.from,
                to: this.rangeDates.to
              };
            } else {
              for (
                let d = new Date(this.rangeDates.from.getTime());
                d.getTime() <= this.rangeDates.to.getTime();
                d.setDate(d.getDate() + 1)
              ) {
                if (this.isInRange(d)) pickerValue.push(new Date(d.getTime()));
              }
            }
            this.pickerValue = pickerValue;
            if (this.closeOnClick) this.closeCalendar();
          }
        } else {
          if (this.closeOnClick) this.closeCalendar();
          this.pickerValue = this.selectedDate;
        }
      }

      this.$emit("input", this.pickerValue)
    },

    zeroPadNumber(number) {
      return number.toString().padStart(2, "0");
    },

    formatDate(date) {
      if (date) {
        let day = date.getDate();
        let month = date.getMonth() + 1;
        let year = date.getFullYear();

        if (this.zeroPad) {
          day = this.zeroPadNumber(day);
          month = this.zeroPadNumber(month);
        }

        let returnDate = this.format
          .replace(/%d/g, day)
          .replace(/%m/g, month)
          .replace(/%Y/g, year)
          .replace(/%y/g, year % 100);
        return returnDate;
      }
    },

    openSelectors(calendar) {
      calendar.isSelectMonthOpened = true;
    },

    toggleCalendar() {
      this.isCalendarOpened = !this.isCalendarOpened;
    },

    closeCalendar() {
      if (!this.opened) this.isCalendarOpened = false;
    },

    findFirstDay(week) {
      return week.find(d => d.getDate() === 1);
    },

    compareDates(a, b) {
      if (a && b) return this.setAllToZero(a).getTime() === this.setAllToZero(b).getTime();
      return false;
    },

    setAllToZero(date){
      date.setHours(0);
      date.setMinutes(0);
      date.setSeconds(0);
      date.setMilliseconds(0);
      return date;
    },

    isBetween(date, a, b) {
      if (date && a && b) return date.getTime() >= a && date.getTime() <= b;
      return false;
    },

    isBefore(a, b){
      return this.setAllToZero(a) < this.setAllToZero(b)
    },

    isAfter(a, b){
      return this.setAllToZero(a) > this.setAllToZero(b)
    },

    isDisabled(date) {
      for (let i = 0; i < this.disabledDates.length; i++) {
        if(Array.isArray(this.disabledDates[i])){
          let d = [...this.disabledDates[i]];
          d.sort((a, b) => a - b);
          if (this.isBetween(date, d[0], d[1])) return true;
        } else if(this.compareDates(date, this.disabledDates[i])) return true
      }

      if(this.disableBefore && this.isBefore(date, this.disableBefore)) return true;
      if(this.disableAfter && this.isAfter(date, this.disableAfter)) return true;
      return false;
    },

    isInExactRange(date) {
      if (this.exactRangeDates && Array.isArray(this.pickerValue)) {
        return this.pickerValue.some(d => this.compareDates(d, date));
      }
      return false;
    },

    isInRange(date) {
      if (this.range)
        return (
          !this.isDisabled(date) &&
          (this.isBetween(date, this.rangeDates.from, this.rangeDates.to) ||
            this.isInExactRange(date))
        );
      return false;
    },

    isDifferentMonth(date, week) {
      if (this.fillEmpty)
        return date.getMonth() !== this.findFirstDay(week).getMonth();
      return false;
    }
  },

  watch: {
    value: {
      handler(value){
        this.pickerValue = value;
        if(value){
          if(!value.from) this.selectedDate = value
          else {
            this.rangeDates.from = !this.exactRangeDates && value.from ? value.from : undefined;
            this.rangeDates.to = !this.exactRangeDates && value.to ? value.to : undefined
            this.selectedDate = undefined;
          }
        }
        this.$emit("input", value);
      },
      deep: true
    }
  }
};
</script>
<style lang="scss">
.calendar {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  * {
    box-sizing: border-box;
  }

  &.isOpened{
    z-index: 1000;
  }

  &.isFillRangeEnabled {
    .calendar__Day {
      position: relative;
      &.isInRange {
        &:before {
          position: absolute;
          content: "";
          top: 0;
          left: -2px;
          right: -2px;
          bottom: 0;
          z-index: -1;
          background-color: inherit;
        }

        &.isFirstInRange:before{
          left: 0;
          border-top-left-radius: 50%;
          border-bottom-left-radius: 50%;
        }

        &.isLastInRange:before{
          right: 0;
          border-top-right-radius: 50%;
          border-bottom-right-radius: 50%;
        }
      }
    }
  }

  &.isAllwaysOpened {
    .calendar__Items {
      position: relative;
    }

    .calendar__Input {
      cursor: default;
    }
  }
}

.calendar__Week,
.calendar__DayNames {
  display: flex;
}

.calendar__Day,
.calendar__DayName {
  min-width: 28px;
  height: 28px;
  margin: 2px;
  font-size: 14px;
  text-align: center;
}

.calendar__DayName {
  font-weight: bold;
}

.calendar__Day {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  cursor: pointer;
  &:hover {
    &:not(.isSelected):not(.isInRange) {
      transition: background-color 200ms, color 200ms;
      background-color: #eee;
      color: #000;
    }
  }
}

.calendar__Day {
  &.isToday {
    background-color: #aaa;
    color: #fff;
  }

  &.isSelected,
  &.isInRange {
    background-color: #333;
    color: #fff;
  }

  &.isDisabled {
    opacity: 0.5;
    pointer-events: none;
  }

  &.isDifferentMonth {
    opacity: 0.25;
  }
}

.calendar__Input {
  cursor: pointer;
}

.calendar__Header {
  display: flex;
  padding-bottom: 20px;
  text-align: center;
  font-size: 16px;
  justify-content: space-between;
}

.calendar__HeaderDate {
  cursor: pointer;
  margin: 0 auto;
}

.calendar__Items {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  background-color: #fff;
  border: 1px solid #333;
  padding: 20px 0 0;
}

.calendar__Item {
  position: relative;
  margin: 0 20px 20px;
}

.calendar__Months,
.calendar__Years {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 10;
  background-color: #111;
  ul {
    display: flex;
    flex-wrap: wrap;
    list-style-type: none;
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
  }

  li {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 25%;
    color: #fff;
    cursor: pointer;
    transition: background-color 200ms;
    &:hover {
      background-color: #555;
    }

    &.isSelected {
      background-color: #333;
    }
  }
}

.calendar__Months {
  li {
    width: 33.333%;
    height: 25%;
  }
}

.calendar__Years {
  padding-top: 40px;
  align-items: flex-end;
  li {
    width: 33.333%;
    height: 33.333%;
  }

  button {
    position: absolute;
    top: 10px;
    &:first-of-type {
      left: 10px;
    }

    &:last-of-type {
      right: 10px;
    }

    &:hover {
      opacity: 0.5;
    }

    span {
      background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' version='1.1' viewBox='0 0 129 129' enable-background='new 0 0 129 129' width='512px' height='512px'%3E%3Cg%3E%3Cpath xmlns='http://www.w3.org/2000/svg' fill='%23fff' d='m88.6,121.3c0.8,0.8 1.8,1.2 2.9,1.2s2.1-0.4 2.9-1.2c1.6-1.6 1.6-4.2 0-5.8l-51-51 51-51c1.6-1.6 1.6-4.2 0-5.8s-4.2-1.6-5.8,0l-54,53.9c-1.6 1.6-1.6,4.2 0,5.8l54,53.9z'/%3E%3C/g%3E%3C/svg%3E");
    }
  }
}

.calendar__Button {
  position: relative;
  cursor: pointer;
  background-color: transparent;
  border: 0;
  outline: 0;
  padding: 5px;
  width: 20px;
  height: 20px;
  transition: opacity 200ms;
  &:hover {
    opacity: 0.5;
  }

  span {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    font-size: 0;
    background-size: cover;
    background-repeat: no-repeat;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' version='1.1' viewBox='0 0 129 129' enable-background='new 0 0 129 129' width='512px' height='512px'%3E%3Cg%3E%3Cpath xmlns='http://www.w3.org/2000/svg' fill='%23000' d='m88.6,121.3c0.8,0.8 1.8,1.2 2.9,1.2s2.1-0.4 2.9-1.2c1.6-1.6 1.6-4.2 0-5.8l-51-51 51-51c1.6-1.6 1.6-4.2 0-5.8s-4.2-1.6-5.8,0l-54,53.9c-1.6 1.6-1.6,4.2 0,5.8l54,53.9z'/%3E%3C/g%3E%3C/svg%3E");
  }
}

.calendar__Next {
  span {
    transform: rotate(180deg);
  }
}
</style>