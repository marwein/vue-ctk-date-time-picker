<template>
  <transition
    :name="this.agendaPosition === 'top' ? 'slide' : 'slideinvert'"
  >
    <div
      class="datetimepicker flex"
      :class="{'inline': withoutInput}"
      v-show="visible || withoutInput"
      @click.stop
      :style="position"
    >
      <div class="datepicker" :style="position">
        <div class="datepicker-header" :style="bgStyle" v-if="withoutHeader">
          <div class="datepicker-year" v-if="!disableDate">
            <transition-group :name="transitionDayName" >
              <div v-for="year in [year]" :key="year">{{year}}</div>
            </transition-group>
          </div>
          <div class="flex justify-content-between">
            <transition-group :name="transitionDayName" class="datepicker-date dots-text flex-1" v-if="!disableDate">
              <span v-for="dateFormatted in [getDateFormatted()]" :key="dateFormatted">{{ getDateFormatted() }}</span>
            </transition-group>
            <div class="datepicker-time flex justify-content-center" v-if="!disableTime && !isFormatTwelve" :style="timeWidth">
              <transition-group :name="transitionDayName" class="dots-text datepicker-hour flex-1 flex justify-content-right">
                <span v-for="hour in [dateTime.format('HH')]" :key="hour">{{ hour }}</span>
              </transition-group>
              <span>:</span>
              <transition-group :name="transitionDayName" class="dots-text datepicker-minute flex-1 flex justify-content-left">
                <span v-for="min in [dateTime.format('mm')]" :key="min">{{ min }}</span>
              </transition-group>
            </div>
            <div class="datepicker-time flex justify-content-center" v-else-if="!disableTime" :style="timeWidth">
              <transition-group :name="transitionDayName" class="dots-text datepicker-hour flex-1 flex justify-content-center">
                <span v-for="hour in [dateTime.format(timeFormat)]" :key="hour">{{ hour }}</span>
              </transition-group>
            </div>
          </div>
        </div>
        <div class="datetimepicker-container flex">
          <ctk-date-picker
            :without-input="withoutInput"
            :no-weekends-days="noWeekendsDays"
            :month="month"
            :date-time="dateTime"
            :locale="locale"
            :color="color"
            @change-date="selectDate"
            @change-month="changeMonth"
            v-if="!disableDate"
            :min-date="minDate"
            :max-date="maxDate"
            :value="value"
          />
          <ctk-time-picker
            ref="timePickerComponent"
            :month="month"
            :date-time="dateTime"
            :color="color"
            :format="timeFormat"
            :minute-interval="minuteInterval"
            v-if="!disableTime"
            :visible="visible"
            @change-time="selectTime"
            :value="value"
          />
        </div>
        <div class="datepicker-buttons-container flex justify-content-right" v-if="enableButtonValidate && !withoutInput && !autoClose">
          <button tabindex="-1" class="datepicker-button validation flex align-center justify-content-center" @click="validate">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path d="M0 0h24v24H0z" fill="none"/>
              <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
            </svg>
            <span class="datepicker-button-effect"></span>
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import CtkTimePicker from './_subs/CtkTimePicker.vue'
import CtkDatePicker from './_subs/CtkDatePicker.vue'
import Month from './../modules/month'
import moment from 'moment'
export default {
  name: 'CtkDatePickerAgenda',
  components: {
    CtkTimePicker, CtkDatePicker
  },
  props: {
    dateTime: {},
    visible: { type: Boolean, required: true, default: true },
    disableTime: { type: Boolean },
    disableDate: { type: Boolean },
    minuteInterval: { type: Number },
    color: { type: String },
    timeFormat: { type: String },
    withoutHeader: {},
    locale: {},
    maxDate: {},
    minDate: {},
    withoutInput: {},
    agendaPosition: {},
    noWeekendsDays: {},
    autoClose: {},
    enableButtonValidate: {},
    value: {}
  },
  data () {
    return {
      month: this.disableDate ? '' : new Month(this.dateTime.month(), this.dateTime.year()),
      transitionDayName: 'slidevnext',
      timeWidth: !this.disableTime ? this.dateTimeWidth() : null
    }
  },
  computed: {
    position () {
      if (this.agendaPosition === 'top') {
        return {
          top: '100%',
          marginBottom: '10px'
        }
      } else {
        return {
          bottom: '100%',
          marginTop: '10px'
        }
      }
    },
    isFormatTwelve () {
      if (this.timeFormat) {
        return (this.timeFormat.indexOf('a') > -1) || (this.timeFormat.indexOf('A') > -1)
      } else {
        return false
      }
    },
    bgStyle () {
      return {
        backgroundColor: this.color,
        padding: this.disableDate ? '10px 0' : '10px 0 10px 10px'
      }
    },
    year () {
      return this.dateTime.format('YYYY')
    }
  },
  methods: {
    getDateFormatted () {
      return moment(this.dateTime).locale(this.locale).format('ddd D MMM')
    },
    selectTime (dateTime) {
      this.transitionDayName = 'slidevprev'
      if (dateTime > this.dateTime) {
        this.transitionDayName = 'slidevnext'
      }
      this.$emit('change-date', dateTime)
    },
    selectDate (dateTime) {
      this.transitionDayName = 'slidevnext'
      if (dateTime.isBefore(this.dateTime)) {
        this.transitionDayName = 'slidevprev'
      }
      dateTime.add(this.dateTime.hour(), 'hours')
      dateTime.add(this.dateTime.minute(), 'minutes')
      this.$emit('change-date', dateTime)
    },
    changeMonth (val) {
      let month = this.month.month + (val === 'prev' ? -1 : +1)
      let year = this.month.year
      if (month > 11 || month < 0) {
        year += (val === 'prev' ? -1 : +1)
        month = (val === 'prev' ? 11 : 0)
      }
      this.month = new Month(month, year)
    },
    validate () {
      this.$emit('validate')
    },
    dateTimeWidth () {
      let width
      let result
      if (this.$refs.timePickerComponent && this.$refs.timePickerComponent.$el.clientWidth) {
        width = this.$refs.timePickerComponent.$el.clientWidth
      } else {
        width = 160
      }
      result = {
        flex: '0 0 ' + width + 'px',
        width: width + 'px',
        minWidth: width + 'px',
        maxWidth: width + 'px'
      }
      return result
    }
  },
  watch: {
    dateTime: {
      handler () {
        this.month = this.disableDate ? '' : new Month(this.dateTime.month(), this.dateTime.year())
        this.getDateFormatted()
      },
      deep: true
    },
    locale () {
      this.month = this.disableDate ? '' : new Month(this.dateTime.month(), this.dateTime.year())
      this.getDateFormatted()
    },
    visible (val) {
      if (val && !this.disableTime) {
        this.$nextTick(() => {
          this.timeWidth = this.dateTimeWidth()
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  @import url('https://fonts.googleapis.com/css?family=Roboto:400,500,700');
  @import "../assets/animation.scss";
  .datetimepicker {
    position: absolute;
  }
  .datepicker {
    font-family: 'Roboto', sans-serif;
    position: absolute;
    z-index: 5;
    border-radius: 4px;
    background: #FFF;
    -webkit-box-shadow: 0 2px 1px -1px rgba(0, 0, 0, 0.2), 0 1px 1px 0 rgba(0, 0, 0, 0.14), 0 1px 3px 0 rgba(0, 0, 0, 0.12);
    box-shadow: 0 2px 1px -1px rgba(0, 0, 0, 0.2), 0 1px 1px 0 rgba(0, 0, 0, 0.14), 0 1px 3px 0 rgba(0, 0, 0, 0.12);
    max-width: 400px;
    .datetimepicker-container {
      background: #FFF;
      border-bottom-left-radius: 4px;
      border-bottom-right-radius: 4px;
    }
    &-header {
      background: dodgerblue;
      color: #FFF;
      padding: 10px 0 10px 10px;
      position: relative;
      border-bottom: 1px solid #EAEAEA;
      border-top-left-radius: 4px;
      border-top-right-radius: 4px;
      .datepicker-year {
        opacity: 0.7;
        margin-bottom: 10px;
        font-size: 14px;
        line-height: 14px;
        position: relative;
        height: 14px;
      }
      .datepicker-date, .datepicker-time, .datepicker-minute, .datepicker-hour {
        font-size: 20px;
        line-height: 20px;
        position: relative;
        height: 20px;
      }
      .datepicker-date {
        text-transform: capitalize;
      }
    }
    .datepicker-buttons-container {
      padding: 5px 10px;
      border-top: 1px solid #EAEAEA;
      .datepicker-button {
        cursor: pointer;
        height: 35px;
        width: 35px;
        border: none;
        outline: none;
        appearance: none;
        border-radius: 50%;
        padding: 0;
        position: relative;
        svg {
          position: relative;
          -webkit-transition: all 450s cubic-bezier(0.23, 1, 0.32, 1) 0ms;
          transition: all 450s cubic-bezier(0.23, 1, 0.32, 1) 0ms;
        }
        .datepicker-button-effect {
          position: absolute;
          opacity: 0.6;
          height: 30px;
          width: 30px;
          top: 2px;
          left: 2px;
          border-radius: 50%;
          -webkit-transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
          transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;
          transform: scale(0);
        }
        &.validation {
          svg {
            fill: green;
            position: relative;
          }
          .datepicker-button-effect {
            background: green;
          }
          &:hover {
            .datepicker-button-effect {
              transform: scale(1);
              opacity: 0.6;
            }
            svg {
              fill: white !important;
            }
          }
        }
      }
    }
  }
  .inline {
    &.datetimepicker, .datepicker {
      position: relative;
    }
    .datepicker {
      margin-bottom: 0 !important;
      box-shadow: none;
      -webkit-box-shadow: none;
      width: 100%;
      max-width: 100%;
    }
  }
  @media screen and (max-width: 412px) {
    .datetimepicker-container {
      -webkit-flex-direction: column;
      -ms-flex-direction: column;
      flex-direction: column;
      flex-flow: column;
      -moz-flex-direction: column;
    }
    .datetimepicker:not(.inline) {
      margin: 0 !important;
      position: absolute;
      top: 0 !important;
      bottom: 0;
      right: 0;
      left: 0;
      .datepicker {
        top: 50px !important;
        left: 5%;
        width: 90%;
        max-width: inherit;
        position: fixed;
        margin: 0 !important;
      }
      .datepicker-header {
        padding: 10px !important;
        .datepicker-time {
          min-width: 100px !important;
          max-width: 100px !important;
        }
        .datepicker-time, .datepicker-time .flex-1 {
          justify-content: flex-end;
          -ms-flex-pack: end;
          -moz-box-align: end;
          -moz-box-pack: end;
          -webkit-box-pack: end;
          -webkit-justify-content: flex-end;
          -webkit-box-align: end;
        }
      }
    }
  }
</style>
