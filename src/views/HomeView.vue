<template>
  <div class="home">
    <p>Google Calendar API Quickstart</p>

    <button
      class="btn btn-primary"
      id="authorize_button"
      @click="handleAuthClick()"
    >
      Login</button
    ><br />
    <button id="signout_button" @click="handleSignoutClick()">Sign Out</button>
    <form @submit.prevent>
      <div class="form-group">
        <label for="exampleInputEmail1">Summary</label>
        <input
          class="form-control"
          id="exampleInputEmail1"
          aria-describedby="emailHelp"
          placeholder="summary"
          v-model="summary"
        />
      </div>
      <div class="form-group">
        <label>Location of event</label>
        <input class="form-control" placeholder="Location" v-model="location" />
      </div>
      <div class="form-group">
        <label>Description of event</label>
        <input
          class="form-control"
          placeholder="description"
          v-model="description"
        />
      </div>
      <div class="form-group">
        <label for="exampleInputPassword1">Attendees of event</label>
        <div style="display: flex">
          <input
            class="form-control"
            placeholder="Attendee's email"
            v-model="tempAttendee"
          />
          <button @click="addAttendee()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-person-plus-fill"
              viewBox="0 0 16 16"
            >
              <path
                d="M1 14s-1 0-1-1 1-4 6-4 6 3 6 4-1 1-1 1H1zm5-6a3 3 0 1 0 0-6 3 3 0 0 0 0 6z"
              />
              <path
                fill-rule="evenodd"
                d="M13.5 5a.5.5 0 0 1 .5.5V7h1.5a.5.5 0 0 1 0 1H14v1.5a.5.5 0 0 1-1 0V8h-1.5a.5.5 0 0 1 0-1H13V5.5a.5.5 0 0 1 .5-.5z"
              />
            </svg>
          </button>
        </div>
        <div v-show="attendees.length !== 0">
          <p>Invited people are:-</p>
          <ol v-for="people in attendees" :key="people">
            {{
              people.email
            }}
          </ol>
        </div>
        <div class="form-group">
          <label>Start event on</label>
          <input
            class="form-control"
            v-model="startDateTime"
            type="datetime-local"
            id="datepicker"
          />
        </div>
        <div class="form-group">
          <label>End event on</label>
          <input
            class="form-control"
            v-model="endDateTime"
            type="datetime-local"
            id="datepicker"
          />
        </div>
      </div>
      <button @click="create()" class="btn btn-primary">Create Event</button>
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
      startDateTime: "",
      endDateTime: "",
      tempAttendee: "",
      summary: "",
      location: "",
      description: "",
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
      
      const event = {
        summary: this.summary,
        location: this.location,
        description: this.description,
        // start: this.start,
        start: {
          dateTime: new Date(this.startDateTime).toISOString(),
          timeZone: "America/Los_Angeles",
        },
        end: {
          dateTime: new Date(this.endDateTime).toISOString(),
          timeZone: "America/Los_Angeles",
        },
        // recurrence: ["RRULE:FREQ=DAILY;COUNT=2"],
        attendees: [
          ...this.attendees,
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
      this.handleAuthClick();
      this.attendees = [];
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
