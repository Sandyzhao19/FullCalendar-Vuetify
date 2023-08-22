<script lang='ts'>
import {defineComponent, ref} from 'vue'
import bootstrap5Plugin from '@fullcalendar/bootstrap5';
import {CalendarOptions, EventApi, DateSelectArg, EventClickArg} from '@fullcalendar/core'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'
import {INITIAL_EVENTS, createEventId} from './event-utils'
import listPlugin from '@fullcalendar/list';

export default defineComponent({
  components: {
    FullCalendar,
  },
  data() {
    return {
      calendarView: 'Month',
      calendarTitle: '',

      calendarOptions: {
        plugins: [
          dayGridPlugin,
          timeGridPlugin,
          interactionPlugin,
          listPlugin
        ],
        headerToolbar: false,

        initialView: 'dayGridMonth',
        initialEvents: INITIAL_EVENTS,
        editable: true,
        selectable: true,
        selectMirror: true,
        dayMaxEvents: true,
        weekends: true,
        select: this.handleDateSelect,
        eventClick: this.handleEventClick,
        eventsSet: this.handleEvents,
        /* you can update a remote database when these fire:
        eventAdd:
        eventChange:
        eventRemove:
        */
      } as CalendarOptions,
      currentEvents: [] as EventApi[],
    }
  },
  mounted() {
    this.calendarTitle = this.$refs.fullCalendar?.getApi().view.title;
  },
  methods: {
    handleWeekendsToggle() {
      this.calendarOptions.weekends = !this.calendarOptions.weekends;
    },
    handleDateSelect(selectInfo: DateSelectArg) {
      let title = prompt('Please enter a new title for your event');
      let calendarApi = selectInfo.view.calendar;
      
      calendarApi.unselect() // clear date selection

      if (title) {
        calendarApi.addEvent({
          id: createEventId(),
          title,
          start: selectInfo.startStr,
          end: selectInfo.endStr,
          allDay: selectInfo.allDay
        })
      }
    },
    handleEventClick(clickInfo: EventClickArg) {
      if (confirm(`Are you sure you want to delete the event '${clickInfo.event.title}'`)) {
        clickInfo.event.remove();
      }
    },
    handleEvents(events: EventApi[]) {
      this.currentEvents = events;
    },

    changePrev() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
      calendarInstance.getApi().prev();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeNext() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
      calendarInstance.getApi().next();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeToToday() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
      calendarInstance.getApi().today();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeCalendarView() {
      if (this.calendarView === 'Month') {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
        calendarInstance.getApi().changeView('dayGridMonth');
        this.calendarTitle = calendarInstance.getApi().view.title;

      } else if (this.calendarView === 'Week') {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
        calendarInstance.getApi().changeView('timeGridWeek');
        this.calendarTitle = calendarInstance.getApi().view.title;

      } else if (this.calendarView === 'Day') {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
        calendarInstance.getApi().changeView('timeGridDay');
        this.calendarTitle = calendarInstance.getApi().view.title;

      }
    },

    changeListWeek() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<typeof FullCalendar>;
      calendarInstance.getApi().changeView('listWeek');
      this.calendarTitle = calendarInstance.getApi().view.title;
    },
  }
})


</script>

<template>
  <div class='demo-app'>
    <div class='demo-app-main'>
      <v-row justify="space-between" class="align-center mb-3">
        <v-col cols="12" md="4" class="d-flex align-center justify-center">
          <v-btn icon variant="text" @click="changePrev">
            <ChevronLeftIcon size="20"/>
          </v-btn>
          <v-btn icon variant="text">
            <ChevronRightIcon size="20" @click="changeNext"/>
          </v-btn>
          <v-btn color="primary" variant="outlined" @click="changeToToday">Today</v-btn>
          <v-select class="ml-3 calendar-view-select" color="primary" :items="['Month', 'Week', 'Day']"
                    variant="outlined"
                    v-model="calendarView" hide-details @update:modelValue="changeCalendarView"
                    density="compact"></v-select>
        </v-col>

        <v-col cols="12" md="5" class="d-flex align-center justify-center">
          <h4 class="text-h3">{{ calendarTitle }}</h4>
        </v-col>

        <v-col cols="12" md="3">
          <div class="d-flex gap-2 justify-end">
            <v-btn color="primary" icon variant="text" @click="changeListWeek">
              <ListIcon size="20"/>
            </v-btn>
            <v-btn color="primary" icon variant="text">
              <LayoutGridIcon size="20" @click="changeCalendarView"/>
            </v-btn>
          </div>
        </v-col>
      </v-row>

      <FullCalendar
          class='demo-app-calendar'
          :options='calendarOptions'
          ref='fullCalendar'
      >
        <template v-slot:eventContent='arg'>
          <b>{{ arg.timeText }}</b>
          <i>{{ arg.event.title }}</i>
        </template>

      </FullCalendar>
    </div>
  </div>
</template>

<style lang='css'>
h2 {
  margin: 0;
  font-size: 16px;
}

ul {
  margin: 0;
  padding: 0 0 0 1.5em;
}

li {
  margin: 1.5em 0;
  padding: 0;
}

b { /* used for event dates/times */
  margin-right: 3px;
}

.demo-app {
  display: flex;
  min-height: 100%;
  font-family: Arial, Helvetica Neue, Helvetica, sans-serif;
  font-size: 14px;
}

.demo-app-sidebar {
  width: 300px;
  line-height: 1.5;
  background: #eaf9ff;
  border-right: 1px solid #d3e2e8;
}

.demo-app-sidebar-section {
  padding: 2em;
}

.demo-app-main {
  flex-grow: 1;
  padding: 3em;
}

.fc { /* the calendar root */
  margin: 0 auto;
}

.calendar-view-select .v-field {
  max-width: 150px;
}
</style>
