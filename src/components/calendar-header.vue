<template>
  <div class="calendar-header">
    <div class="header-left">
      <slot name="header-left">
          <button class="btn btn-outline-primary btn-sm" @click.stop="goToday"><i class="fal fa-calendar-star"></i> </button>
      </slot>
    </div>
    <div class="header-center">

      <span class="title"> {{ month }} {{ year }} </span>

    </div>
    <div class="header-right">
      <slot name="header-right">

          <button class="btn btn-outline-primary btn-sm prev-month" v-if="!isPrevMonthDisabled" @click.stop="goPrev"><i class="fal fa-chevron-left"></i></button>
          <button class="btn btn-outline-primary btn-sm next-month" v-if="!isNextMonthDisabled" c @click.stop="goNext"><i class="fal fa-chevron-right"></i></button>

      </slot>
    </div>
  </div>
</template>

<script>
  import i18nMixin from '../mixins/i18n';
  import calendarJs from '../utils/calendar';

  export default {
    name: 'calendar-header',
    mixins: [ i18nMixin ],
    props: {
      disable: {
        type: Object,
        required: true
      }
    },
    data() {
      return {
        monthStart: null
      };
    },
    computed: {
      year() {
        return this.monthStart.getFullYear();
      },
      month() {
        return this.printMonth(this.monthStart.getMonth());
      },
      hasDisabledPeriod() {
        return !Object.keys(this.disable).length;
      },
      isPrevMonthDisabled() {
        if (this.hasDisabledPeriod || !this.disable.hasOwnProperty('to')) {
          return false;
        }

        return (this.disable.to.getMonth() >= this.monthStart.getMonth()) &&
               (this.disable.to.getFullYear() >= this.monthStart.getFullYear());
      },
      isNextMonthDisabled() {
        if (this.hasDisabledPeriod || !this.disable.hasOwnProperty('from')) {
          return false;
        }

        return (this.disable.from.getMonth() <= this.monthStart.getMonth()) &&
               (this.disable.from.getFullYear() <= this.monthStart.getFullYear());
      }
    },
    methods: {
      goPrev() {
        if (!this.previousMonthDisabled) {
          this.monthStart = calendarJs.shiftMonth(this.monthStart, 1);
        }
      },
      goNext() {
        if (!this.nextMonthDisabled) {
          this.monthStart = calendarJs.shiftMonth(this.monthStart, -1);
        }
      },
      goToDate(date) {
        this.monthStart = calendarJs.firstDateOfMonth(date);
      },
      goToday(){
        var date = new Date();
        var firstDay = new Date(date.getFullYear(), date.getMonth(), 1);
        this.monthStart = firstDay
      }
    },
    watch: {
      monthStart(monthStart) {
        const monthEnd = calendarJs.lastDateOfMonth(monthStart);

        this.$calendar.eventBus.$emit('month-changed',
          monthStart,
          monthEnd
        );
      }
    },
    created() {
      this.monthStart = calendarJs.firstDateOfMonth();
      this.$calendar.eventBus.$on('go-to-date', (date) => this.goToDate(date));
    }
  }
</script>

<style lang="scss">
  .calendar-header{
    align-items: center;
  }
  .header-left, .header-right{
    flex:1;
  }
  .header-center {
    flex: 3;
    text-align: center;
  }
  .title{
    margin: 0 5px;
  }
  .prev-month, .next-month{
    cursor: pointer;
  }
</style>
