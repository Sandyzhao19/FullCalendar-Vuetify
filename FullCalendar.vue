<script lang="ts">
import { defineComponent, ref } from "vue";
import bootstrap5Plugin from "@fullcalendar/bootstrap5";
import {
  CalendarOptions,
  EventApi,
  DateSelectArg,
  EventClickArg,
} from "@fullcalendar/core";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import { INITIAL_EVENTS, createEventId } from "./event-utils";
import listPlugin from "@fullcalendar/list";

export default defineComponent({
  components: {
    FullCalendar,
  },
  data() {
    return {
      calendarTitle: "",

      calendarOptions: {
        plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin, listPlugin],
        headerToolbar: false,

        initialView: "dayGridMonth",
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
    };
  },
  mounted() {
    this.$nextTick(() => {
      // @ts-ignore
      if (this.$refs.fullCalendar) {
        // @ts-ignore
        const calendarInstance = this.$refs.fullCalendar.getApi();
        this.calendarTitle = calendarInstance.view.title;
      }
    });
  },
  methods: {
    handleWeekendsToggle() {
      this.calendarOptions.weekends = !this.calendarOptions.weekends;
    },
    handleDateSelect(selectInfo: DateSelectArg) {
      let title = prompt("Please enter a new title for your event");
      let calendarApi = selectInfo.view.calendar;

      calendarApi.unselect(); // clear date selection

      if (title) {
        calendarApi.addEvent({
          id: createEventId(),
          title,
          start: selectInfo.startStr,
          end: selectInfo.endStr,
          allDay: selectInfo.allDay,
        });
      }
    },
    handleEventClick(clickInfo: EventClickArg) {
      if (
        confirm(
          `Are you sure you want to delete the event '${clickInfo.event.title}'`
        )
      ) {
        clickInfo.event.remove();
      }
    },
    handleEvents(events: EventApi[]) {
      this.currentEvents = events;
    },

    changePrev() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<
        typeof FullCalendar
      >;
      calendarInstance.getApi().prev();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeNext() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<
        typeof FullCalendar
      >;
      calendarInstance.getApi().next();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeToToday() {
      const calendarInstance = this.$refs.fullCalendar as InstanceType<
        typeof FullCalendar
      >;
      calendarInstance.getApi().today();
      this.calendarTitle = calendarInstance.getApi().view.title;
    },

    changeCalendarView(viewStr: string) {
      if (viewStr === "Month") {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<
          typeof FullCalendar
        >;
        calendarInstance.getApi().changeView("dayGridMonth");
        this.calendarTitle = calendarInstance.getApi().view.title;
      } else if (viewStr === "Week") {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<
          typeof FullCalendar
        >;
        calendarInstance.getApi().changeView("timeGridWeek");
        this.calendarTitle = calendarInstance.getApi().view.title;
      } else if (viewStr === "Day") {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<
          typeof FullCalendar
        >;
        calendarInstance.getApi().changeView("timeGridDay");
        this.calendarTitle = calendarInstance.getApi().view.title;
      } else if (viewStr === "List") {
        const calendarInstance = this.$refs.fullCalendar as InstanceType<
          typeof FullCalendar
        >;
        calendarInstance.getApi().changeView("listWeek");
        this.calendarTitle = calendarInstance.getApi().view.title;
      }
    },
  },
});
</script>

<template>
  <div class="demo-app">
    <div class="demo-app-sidebar">
      <div class="demo-app-sidebar-section">
        <h2>Instructions</h2>
        <ul>
          <li>Select dates and you will be prompted to create a new event</li>
          <li>Drag, drop, and resize events</li>
          <li>Click an event to delete it</li>
        </ul>
      </div>
      <div class="demo-app-sidebar-section">
        <label>
          <input
            type="checkbox"
            :checked="calendarOptions.weekends"
            @change="handleWeekendsToggle"
          />
          toggle weekends
        </label>
      </div>
      <div class="demo-app-sidebar-section">
        <h2>All Events ({{ currentEvents.length }})</h2>
        <ul>
          <li v-for="event in currentEvents" :key="event.id">
            <b>{{ event.startStr }}</b>
            <i>{{ event.title }}</i>
          </li>
        </ul>
      </div>
    </div>
    <div class="demo-app-main">
      <v-row justify="space-between" class="align-center mb-3">
        <v-col cols="12" md="3">
          <v-btn color="primary" variant="outlined" @click="changeToToday"
            >Today</v-btn
          >
        </v-col>

        <v-col cols="12" md="6" class="d-flex align-center justify-center">
          <v-btn icon variant="text" @click="changePrev">
            <ChevronLeftIcon size="20" />
          </v-btn>

          <h4 class="text-h3 mx-5">{{ calendarTitle }}</h4>

          <v-btn icon variant="text">
            <ChevronRightIcon size="20" @click="changeNext" />
          </v-btn>
        </v-col>

        <v-col cols="12" md="3">
          <div class="d-flex gap-2 justify-end">
            <v-btn-group divided variant="outlined" border rounded>
              <v-btn
                color="primary"
                icon
                variant="text"
                @click="changeCalendarView('Month')"
              >
                <LayoutGridIcon stroke-width="1.5" size="20" />
                <v-tooltip activator="parent" location="top">Month</v-tooltip>
              </v-btn>

              <v-btn
                color="primary"
                icon
                variant="text"
                @click="changeCalendarView('Week')"
              >
                <TemplateIcon stroke-width="1.5" size="20" />
                <v-tooltip activator="parent" location="top">Week</v-tooltip>
              </v-btn>

              <v-btn
                color="primary"
                icon
                variant="text"
                @click="changeCalendarView('Day')"
              >
                <LayoutListIcon stroke-width="1.5" size="20" />
                <v-tooltip activator="parent" location="top">Day</v-tooltip>
              </v-btn>

              <v-btn
                color="primary"
                icon
                variant="text"
                @click="changeCalendarView('List')"
              >
                <ListNumbersIcon stroke-width="1.5" size="20" />
                <v-tooltip activator="parent" location="top">List</v-tooltip>
              </v-btn>
            </v-btn-group>
          </div>
        </v-col>
      </v-row>

      <FullCalendar
        class="demo-app-calendar"
        :options="calendarOptions"
        ref="fullCalendar"
      >
        <template v-slot:eventContent="arg">
          <b>{{ arg.timeText }}</b>
          <i>{{ arg.event.title }}</i>
        </template>
      </FullCalendar>
    </div>
  </div>
</template>

<style lang="css">
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

b {
  /* used for event dates/times */
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

.fc {
  /* the calendar root */
  margin: 0 auto;
}

.calendar-view-select .v-field {
  max-width: 150px;
}
</style>
