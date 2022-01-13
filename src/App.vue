<template>
  <v-app id="ics-viewer" v-cloak>
    <ds-calendar-app ref="app" :read-only="true">
      <template slot="title"> {{ title }} </template>
      <template slot="eventPopover" slot-scope="slotData"></template>
    </ds-calendar-app>
  </v-app>
</template>

<script>
import axios from "axios";
import ICAL from "ical.js";
import { Parse } from "dayspan";

export default {
  name: "app",

  data: () => ({
    title: "Calendar",
  }),

  mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    const ics =
      urlParams.get("ics") ||
      prompt("Please enter a URL to your ICS/ICAL file");
    this.title =
      urlParams.get("title") ||
      prompt("Please enter a Title for your calendar");
    window.document.title = this.title;

    const search = `?title=${this.title}&ics=${ics}`;
    if (window.location.search != search) {
      window.location.search = search;
    }

    const url = "https://larrybolt-cors-anywhere.herokuapp.com/" + ics;
    axios.defaults.headers["Access-Control-Allow-Origin"] = "*";
    axios.defaults.headers["Content-Type"] =
      "application/x-www-form-urlencoded";
    axios.get(url).then((response) => {
      const vcalendar = new ICAL.Component(ICAL.parse(response.data));
      const events = Array.from(
        vcalendar.getAllSubcomponents("vevent"),
        (vevent) => {
          var event = new ICAL.Event(vevent);
          var start = Parse.day(event.startDate.toString());
          return {
            data: {
              title: event.summary,
              color: "#2196F3",
            },
            schedule: {
              month: [start.month],
              dayOfMonth: [start.dayOfMonth],
              year: [start.year],
              duration: event.duration.days,
              durationUnit: "day",
            },
          };
        }
      );
      this.$refs.app.setState({ events: events });
    });
  },
};
</script>

<style>
body,
html,
#app,
#dayspan {
  font-family: Roboto, sans-serif !important;
  width: 100%;
  height: 100%;
}

.v-btn--flat,
.v-text-field--solo .v-input__slot {
  background-color: #f5f5f5 !important;
  margin-bottom: 8px !important;
}
</style>
