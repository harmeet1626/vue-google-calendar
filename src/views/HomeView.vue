<template>
  <div class="home">
    <h2>Google Calendar API Quickstart</h2>
    <!-- <button @click="test()">test</button> -->
    <div style="display: flex">
      <button
        class="btn btn-primary"
        id="authorize_button"
        @click="handleAuthClick()"
      >
        Login</button
      ><br />
      <button
        class="btn btn-primary"
        id="signout_button"
        @click="handleSignoutClick()"
      >
        Sign Out
      </button>
    </div>
    <div class="top" v-show="isLogin">
      <div class="main">
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
            <label for="location">Location of event</label>
            <input
              class="form-control"
              placeholder="location"
              v-model="location"
              id="location"
            />
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
          <div class="form-group">
            <label for="exampleInputPassword1">Attendees of event</label>
            <div style="display: flex">
              <input
                class="form-control"
                placeholder="Attendee's email"
                v-model="tempAttendee"
              />
              <button class="btn btn light" @click="addAttendee()">
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
              <ol v-for="people in attendees" :key="(i, people)">
                <div class="chip">
                  {{ people.email }}
                  <span class="closebtn" @click="remove_attendee(i)"
                    >&times;</span
                  >
                </div>
              </ol>
            </div>
          </div>
          <br />
          <button @click="create()" class="btn btn-primary">
            Create Event
          </button>
        </form>
      </div>
    </div>
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
        " email profile https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events https://www.googleapis.com/auth/calendar.events.readonly https://www.googleapis.com/auth/calendar.readonly",
      gapiInited: false,
      gisInited: false,
      isLogin: false,
      startDateTime: new Date(),
      endDateTime: new Date(),
      summary: "",
      location: "",
      description: "",
      tempAttendee: "",
      attendees: [],
      reminders: {
        useDefault: false,
        overrides: [
          { method: "email", minutes: 24 * 60 },
          { method: "popup", minutes: 10 },
        ],
      },
      //   tokenClients: "",
      access_token: "",
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
        start: {
          dateTime: new Date(this.startDateTime).toISOString(),
          timeZone: "Asia/Kolkata",
        },
        end: {
          dateTime: new Date(this.endDateTime).toISOString(),
          timeZone: "Asia/Kolkata",
        },
        // recurrence: ["RRULE:FREQ=DAILY;COUNT=2"],
        attendees: [...this.attendees],
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
      this.tokenClient.callback = async (resp) => {
        if (resp.error !== undefined) {
          throw resp;
        }
        document.getElementById("signout_button").style.visibility = "visible";
        document.getElementById("authorize_button").innerText = "Refresh";
        await this.listUpcomingEvents();
        this.isLogin = true;
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
<style scoped>
.chip {
  display: inline-block;
  padding: 0 25px;
  height: 50px;
  font-size: 18px;
  line-height: 50px;
  border-radius: 25px;
  background-color: #f1f1f1;
}
.chip img {
  float: left;
  margin: 0 10px 0 -25px;
  height: 50px;
  width: 50px;
  border-radius: 50%;
}
.closebtn {
  padding-left: 10px;
  color: #888;
  font-weight: bold;
  float: right;
  font-size: 20px;
  cursor: pointer;
}
.closebtn:hover {
  color: #000;
}
.top {
  width: 100%;
  /* height: 100vh; */
  display: flex;
  justify-content: center;
  align-items: center;
}
.main {
  width: 50%;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
  padding: 2%;
}
</style>