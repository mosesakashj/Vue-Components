<template>
  <div style="height: 70%">
    <v-container fluid class="pa-0" v-if="false">
      <v-row>
        <v-col class="d-flex justify-start">
          <v-btn outlined @click="getToday">TODAY</v-btn>
        </v-col>
        <v-col class="d-flex justify-space-around">
          <v-btn icon @click="goPrevious"><v-icon>mdi-chevron-left</v-icon></v-btn>
          <h3 class="mt-1 font-weight-bold">{{ calendarTitle }}</h3>
          <v-btn icon @click="goNext"><v-icon>mdi-chevron-right</v-icon></v-btn>
        </v-col>
        <v-col>
          <v-btn icon @click="toggleWeekends">{{config.weekends ? 'hide Weekends' : 'show weekends'}}</v-btn>
        </v-col>
        <v-col class="d-flex justify-end pb-7">
          <v-select hide-details outlined dense @change="toggleCalendarView" item-text="text" item-value="value" v-model="calendarView" :items="calendarViewItems"></v-select>
        </v-col>
      </v-row>
    </v-container>
    <FullCalendar :key="reInit" ref="calendar" :options="config" v-if="false"></FullCalendar>
  </div>
</template>
<script>
import FullCalendar from '@fullcalendar/vue'
import moment from 'moment'
import resourceTimelinePlugin from '@fullcalendar/resource-timeline'
import resourceDayGridPlugin from '@fullcalendar/resource-daygrid'
import resourceTimeGridPlugin from '@fullcalendar/resource-timegrid'

// import dayGridPlugin from '@fullcalendar/daygrid'
export default {
  data () {
    return{
      config: {
        plugins: [ resourceTimelinePlugin, resourceDayGridPlugin, resourceTimeGridPlugin ],
        initialView: 'resourceTimeGridDay', // resourceTimeline, resourceDayGridDay, resourceTimeGridDay, resourceTimelineWeek
        resources: [
          { title: 'resource a', id: 'a' },
          { title: 'resource b', id: 'b' },
        ],
        weekends: true
      },
      calendarViewItems: [
        { text: 'Month', value: 'resourceTimelineMonth' },
        { text: 'Week', value: 'resourceTimelineWeek'},
        { text: 'Day', value: 'resourceTimeline' }
      ],
      calendarView: 'resourceTimelineMonth',
      calendarTitle: '',
      reInit: 0
    }
  },
  components: {
    FullCalendar
  },
  mounted () {
    this.getCalendarTitle()
  },
  watch: {
    notifyToSearch (value) {
      if (value) {
        this.$_execute('get', `account/query/${ value }`).then(response => {
          if (response.data) this.notifyToItems = response.data
        })
      } else this.notifyToItems = []
    }
  },
  methods: {
    goPrevious () {
      let calendarApi = this.$refs.calendar.getApi();
      calendarApi.prev();
      this.getCalendarTitle()
    },
    goNext () {
      let calendarApi = this.$refs.calendar.getApi();
      calendarApi.next();
      this.getCalendarTitle()
    },
    getToday () {
      let calendarApi = this.$refs.calendar.getApi();
      calendarApi.today();
      this.getCalendarTitle()
    },
    getCalendarTitle () {
      setTimeout(() => {
        let calendarApi = this.$refs.calendar.getApi();
        this.calendarTitle = calendarApi.currentData.viewTitle
      }, 100)
    },
    toggleCalendarView () {
      this.config.initialView = this.calendarView
      this.reInit++
      this.getCalendarTitle()
    },
    dayClicked (date) {
      this.$refs.eventsForm && this.$refs.eventsForm.reset()
      let selectedDate = moment(date).format('YYYY-MM-DD HH:mm:ss')
      this.modelObj.startDate = selectedDate
      this.dialog = true
    },
    eventSelected (event) {
      console.log(event)
    },
    toggleWeekends () {
      this.config.weekends = !this.config.weekends // toggle the boolean!
    }
  }
}
//
</script>
