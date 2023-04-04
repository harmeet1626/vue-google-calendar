<template>
  <div class="home">
    <p>Google Calendar API Quickstart</p>

    <button id="authorize_button" @click="handleAuthClick()">Authorize</button>
    <button id="signout_button" @click="handleSignoutClick()">Sign Out</button>
    <button id="event" @click="create()">Create event</button>
    <form @submit.prevent>
      <input placeholder="summary" v-model="summary" type="text" /><br />
      <input placeholder="Location" v-model="location" /><br />
      <input placeholder="description" v-model="description" /><br />
      <label for="date-input">Choose start date:</label>
      <input
        v-model="startDate"
        type="date"
        id="date-input"
        name="date"
      /><br />
      <label for="time-input">Choose start time:</label>
      <!-- <input
        v-model="startTime"
        type="time"
        id="time-input"
        name="time"
      /><br />
      <label for="date-input">Choose end date:</label>
      <input v-model="endDate" type="date" id="date-input" name="date" /><br />
      <label for="time-input">Choose end time:</label>
      <input v-model="endTime" type="time" id="time-input" name="time" /><br /> -->
      <input v-model="tempAttendee" placeholder="add attendee's email" />
      <button @click="addAttendee()">add attendee</button>
    </form>
    <pre id="content" style="white-space: pre-wrap"></pre>
  </div>
</template>

<script>
export default {
  name: "HomeView",
  data() {
    return {
      CLIENT_ID:
        "197264351142-mtjrbsvctarisoiei3dcaicsltm558h9.apps.googleusercontent.com",

      API_KEY: "AIzaSyCjwXAbJvVuroEIOeVfewCgABFW5vTYe1M",
      DISCOVERY_DOC:
        "https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest",
      SCOPES:
        " https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events https://www.googleapis.com/auth/calendar.events.readonly https://www.googleapis.com/auth/calendar.readonly",

      // tokenClient: null,
      gapiInited: false,
      gisInited: false,
      startDate: "",
      startTime: "",
      endTime: "",
      endDate: "",
      tempAttendee: "",
      summary: "",
      location: "",
      description: "",
      start: {
        dateTime: `${this.startDate}+${this.startTime}`,
        timeZone: "India",
      },
      end: {
        dateTime: `${this.endDate}+${this.endTime}`,
        timeZone: "India",
      },
      attendees: [],
      reminders: {
        useDefault: false,
        overrides: [
          { method: "email", minutes: 24 * 60 },
          { method: "popup", minutes: 10 },
        ],
      },
    };
  },
  mounted() {
    this.gisLoaded();
    this.gapiLoaded();
    document.getElementById("authorize_button").style.visibility = "hidden";
    document.getElementById("signout_button").style.visibility = "hidden";
  },
  methods: {
    addAttendee() {
      this.attendees.push({ email: this.tempAttendee });
      this.tempAttendee = "";
    },
    async create() {
      // const event = {
      //   summary: "Tesing",
      //   location: "Mohali, India",
      //   description: "A chance to hear more about Google's developer products.",
      //   start: {
      //     dateTime: "2023-04-05T09:00:00-07:00",
      //     timeZone: "America/Los_Angeles",
      //   },
      //   end: {
      //     dateTime: "2023-04-05T17:00:00-07:00",
      //     timeZone: "America/Los_Angeles",
      //   },
      //   // recurrence: ["RRULE:FREQ=DAILY;COUNT=2"],
      //   attendees: [
      //     // { email: "nishant_sisodiya@softprodigy.com" },
      //     { email: "sbrin@example.com" },
      //   ],
      //   reminders: {
      //     useDefault: false,
      //     overrides: [
      //       { method: "email", minutes: 24 * 60 },
      //       { method: "popup", minutes: 10 },
      //     ],
      //   },
      // };
      const event = {
        summary: this.summary,
        location: this.location,
        description: this.description,
        start: {
          dateTime: "2023-04-05T09:00:00-07:00",
          timeZone: "America/Los_Angeles",
        },
        end: {
          dateTime: "2023-04-05T17:00:00-07:00",
          timeZone: "America/Los_Angeles",
        },
        // recurrence: ["RRULE:FREQ=DAILY;COUNT=2"],
        attendees: [
          ...this.attendees,
          // { email: "nishant_sisodiya@softprodigy.com" },
          // { email: "sbrin@example.com" },
        ],
        reminders: {
          useDefault: false,
          overrides: [
            { method: "email", minutes: 24 * 60 },
            { method: "popup", minutes: 10 },
          ],
        },
      };
      const request = gapi.client.calendar.events.insert({
        calendarId: "primary",
        resource: event,
      });

      request.execute(function (event) {
        // appendPre('Event created: ' + event.htmlLink);
      });
    },
    gapiLoaded() {
      gapi.load("client", this.initializeGapiClient);
    },
    async initializeGapiClient() {
      await gapi.client.init({
        apiKey: this.API_KEY,
        discoveryDocs: [this.DISCOVERY_DOC],
      });
      this.gapiInited = true;
      this.maybeEnableButtons();
    },
    gisLoaded() {
      this.tokenClient = google.accounts.oauth2.initTokenClient({
        client_id: this.CLIENT_ID,
        scope: this.SCOPES,
        callback: "", // defined later
      });

      this.gisInited = true;
      this.maybeEnableButtons();
    },
    maybeEnableButtons() {
      if (this.gapiInited && this.gisInited) {
        document.getElementById("authorize_button").style.visibility =
          "visible";
      }
    },
    handleAuthClick() {
      // let tokenClient;
      this.tokenClient.callback = async (resp) => {
        if (resp.error !== undefined) {
          throw resp;
        }
        document.getElementById("signout_button").style.visibility = "visible";
        document.getElementById("authorize_button").innerText = "Refresh";
        await this.listUpcomingEvents();
      };

      if (gapi.client.getToken() === null) {
        this.tokenClient.requestAccessToken({ prompt: "consent" });
      } else {
        this.tokenClient.requestAccessToken({ prompt: "" });
      }
    },
    handleSignoutClick() {
      const token = gapi.client.getToken();
      if (token !== null) {
        google.accounts.oauth2.revoke(token.access_token);
        gapi.client.setToken("");
        document.getElementById("content").innerText = "";
        document.getElementById("authorize_button").innerText = "Authorize";
        document.getElementById("signout_button").style.visibility = "hidden";
      }
    },
    async listUpcomingEvents() {
      let response;
      try {
        const request = {
          calendarId: "primary",
          timeMin: new Date().toISOString(),
          showDeleted: false,
          singleEvents: true,
          maxResults: 10,
          orderBy: "startTime",
        };
        response = await gapi.client.calendar.events.list(request);
      } catch (err) {
        document.getElementById("content").innerText = err.message;
        return;
      }

      const events = response.result.items;
      if (!events || events.length == 0) {
        document.getElementById("content").innerText = "No events found.";
        return;
      }
      const output = events.reduce(
        (str, event) =>
          `${str}${event.summary} (${
            event.start.dateTime || event.start.date
          })\n`,
        "Events:\n"
      );
      document.getElementById("content").innerText = output;
    },
  },
};
</script>
