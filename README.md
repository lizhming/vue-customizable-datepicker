# Vue Customizable Datepicker
Try it out here:
[Vue Customizable Datepicker](https://codesandbox.io/s/vue-customizable-datepicker-v48lp)

## 🚀 Usage
```bash
npm i vue-customizable-datepicker
```
[![npm version][npm-version-src]][npm-version-href]
[![npm downloads][npm-downloads-src]][npm-downloads-href]
[![npm downloads][kofi-src]][kofi-href]

## Nuxt.js usage
Create plugin (datepicker.js)
```javascript
import Vue from 'vue'
import DatePicker from 'vue-customizable-datepicker'

Vue.component('DatePicker', DatePicker);

// nuxt.config.js
plugins: [
  {src: '@/plugins/datepicker', ssr: false}
],

// remember to add ssr: false
```
## Slots
### Day
```html
<template v-slot:day="{day}">
  <span>{{ day }}</span>
</template>
```

### Value
```html
<template v-slot:value="{date}">
  <span>{{ date }}</span>
</template>

// If range prop is set to true
<template v-slot:value="{from, to}">
  <span>{{ from }}</span>
  <span>{{ to }}</span>
</template>
```

### Header
```html
<template v-slot:header="{month, year}">
  <span>{{ month }}</span>
  <span>{{ year }}</span>
</template>
```

### Other slots
- arrow-month-next
- arrow-month-previous
- arrow-year-next
- arrow-year-previous

## Events
| Event | Data |
| ------ | ------ |
| year-change | {year: Number, month: Number} |
| month-change | {year: Number, month: Number} |

## Props
| Prop | Default | Description |
| ------ | ------ | ------ |
| arrows | true | ------ |
| closeOnClick | true | ------ |
| dayNames | {1: "Mo",2: "Tu",3: "We",4: "Th",5: "Fr",6: "Sa", 0: "Su"} | ------ |
| disabledDates | [] | ------ |
| disableBefore | Date | ------ |
| disableAfter | Date | ------ |
| exactRangeDates | false | Return array of days |
| fillEmpty | true | ------ |
| fillRange | true | ------ |
| firstDay | 1 | ------ |
| format | true | %d.%m.%Y |
| numberOfCalendars | 1 | ------ |
| opened | false | Always opened |
| placeholder | Pick a date | ------ |
| range | false | Range picker |
| zeroPad | true | Zero pad all except days |
| zeroPadDays | false | Zero pad days |


### disabledDates
```html
<DatePicker :disabled-dates="disabledDates" v-model="selectedDate">
```

```javascript
selectedDate: new Date(),
disabledDates: [
  // To disable range of dates
  [new Date('7/12/2020'), new Date('7/18/2020')],
  
  // To disable single date
  new Date('9/8/2020'),
  
  // example:
  // Disable range from 7/24/2020 to 7/27/2020 + diasble 7/29/2020
  [new Date('7/24/2020'), new Date('7/27/2020')], new Date('7/29/2020')
]
```

## v-model
If **range** prop is set to **true** value will be object with **from** and **to** properties.
```javascript
{
  from: 'Fri Jul 10 2020 00:00:00 GMT+0200 (Central Eu...'
  to: 'Wed Jul 15 2020 00:00:00 GMT+0200 (Central Eu...'
}
```
If **range** and **exactRangeDates** props are set to **true** value will be array of dates.
```javascript
[
  'Thu Jul 09 2020 00:00:00 GMT+0200 (Central Eu...',
  'Fri Jul 10 2020 00:00:00 GMT+0200 (Central Eu...',
  'Sat Jul 11 2020 00:00:00 GMT+0200 (Central Eu...'
]
```
Otherwise value will be **Date** object.
```javascript
"2020-07-08T22:00:00.000Z"
```

### Buy me a coffee
[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F31MWWL)


<!-- Badges -->
[npm-version-src]: https://badgen.net/npm/v/vue-customizable-datepicker/latest
[npm-version-href]: https://npmjs.com/package/vue-customizable-datepicker

[kofi-src]: https://badgen.net/badge/icon/kofi?icon=kofi&label=support
[kofi-href]: https://ko-fi.com/darioferderber

[npm-downloads-src]: https://badgen.net/npm/dm/vue-customizable-datepicker
[npm-downloads-href]: https://npmjs.com/package/vue-customizable