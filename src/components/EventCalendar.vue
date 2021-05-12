<template>
  <div class="collapse" id="navbarToggleExternalContent">
    <div class="bg-dark p-4">
      <h5 class="text-white h4">Collapsed content</h5>
      <span class="text-muted">Toggleable via the navbar brand.</span>
    </div>
  </div>
  <nav class="navbar navbar-dark bg-dark">
    <div class="container-fluid">
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarToggleExternalContent"
        aria-controls="navbarToggleExternalContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
    </div>
  </nav>
  <main role="main" class="container-fluid">
    <div class="container-fluid" id="card-container">
      <div class="card">
        <div class="card-body">
          <h5 class="card-title">Calendar</h5>
          <hr />
          <div class="container-fluid" id="card-body-container">
            <div class="row">
              <div class="col-3">
                <div class="row">
                  <p>Event</p>
                </div>
                <div class="row">
                  <div class="input-group mb-3">
                    <input
                      type="text"
                      class="form-control"
                      v-model="eventName"
                    />
                    <span class="input-group-text" id="basic-addon2">
                      <i class="bi bi-calendar-event"></i>
                    </span>
                  </div>
                </div>
                <div class="row">
                  <div class="col-6">
                    <p>From</p>
                  </div>
                  <div class="col-6">
                    <p>To</p>
                  </div>
                </div>
                <div class="row">
                  <div class="col-6">
                    <input
                      type="date"
                      class="form-control"
                      v-model="eventFrom"
                    />
                  </div>
                  <div class="col-6">
                    <input
                      type="date"
                      class="form-control"
                      v-model="eventTo"
                    />
                  </div>
                </div>
                <div class="row" id="chk-group-days">
                  <div class="col-12">
                    <span v-for="dayOfWeek in daysOfWeek" :key="dayOfWeek.day">
                      <input
                        class="form-check-input"
                        type="checkbox"
                        :id="dayOfWeek.day"
                        :value="dayOfWeek.value"
                        v-model="checkedDays"
                      />
                      <label class="form-check-label" :for="dayOfWeek.day"> {{ dayOfWeek.day }} </label>
                    </span>
                  </div>
                  <div class="row">
                    <div class="input-group mb-3">
                      <button type="button" class="btn btn-primary" @click="saveEvent">
                        Save
                      </button>
                    </div>
                  </div>
                </div>
              </div>
              <div class="col-9">
                <div class="row">
                  <h3>{{ month + " " + year }}</h3>
                  <table class="table">
                    <tbody>
                      <tr
                        v-for="eventDate in eventDates"
                        :key="eventDate.dayOfWeek"
                        :id="eventDate.date"
                        v-bind:class="dateIsInEventDates(eventDate.date) ? 'table-success' : ''"
                      >
                        <td class="col-2">{{ eventDate.dayOfWeek + " " + eventDate.day }}</td>
                        <td><span v-if="dateIsInEventDates(eventDate.date)">{{ eventObj.event_name }}</span></td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import axios from 'axios';
import moment from 'moment';

export default {
  data() {
    return {
      month: "",
      year: "",
      eventDates: [],
      daysOfWeek: [
        {
          'day': 'Mon',
          'value': 1
        },
        {
          'day': 'Tue',
          'value': 2
        },
        {
          'day': 'Wed',
          'value': 3
        },
        {
          'day': 'Thu',
          'value': 4
        },
        {
          'day': 'Fri',
          'value': 5
        },
        {
          'day': 'Sat',
          'value': 6
        },
        {
          'day': 'Sun',
          'value': 0
        },
      ],
      eventName: '',
      eventFrom: '',
      eventTo: '',
      checkedDays: [],
      eventObj: {
        id: '',
        event_name: '',
        event_dates: []
      }
    };
  },
  created() {
    this.getEvent();
    this.getMonthYear();
    this.populateCalendar();
  },
  methods: {
    getMonthYear() {
      var dt = new Date();
      this.month = new Intl.DateTimeFormat("en", { month: "long" }).format(dt);
      this.year = new Intl.DateTimeFormat("en", { year: "numeric" }).format(dt);
    },

    populateCalendar() {
      var dt = new Date();
      var month = dt.getMonth() + 1;
      var year = dt.getFullYear();

      var daysInMonth = new Date(year, month, 0).getDate();

      for (var x = 1; x <= daysInMonth; x++) {
        var strDate = year + "-" + month + "-" + x;
        this.eventDates.push({
          dayOfWeek: x,
          date: moment(strDate).format('YYYY-MM-DD'),
          day: new Intl.DateTimeFormat("en", { weekday: "short" })
                .format(new Date(year, month, x)
          )
        });
      }
    },

    saveEvent() {
      axios.post('http://localhost:8000/api/events', {
        event_name: this.eventName,
        event_date_from: this.eventFrom,
        event_date_to: this.eventTo,
        event_days: this.checkedDays
      })
      .then((response) => {
        this.eventObj = response.data;
      },
      (error) => {
        console.log(error);
      });
    },

    getEvent() {
      axios.get('http://localhost:8000/api/events')
      .then((response) => {
        this.eventObj = response.data;
      },
      (error) => {
        console.log(error);
      });
    },

    dateIsInEventDates(date) {
      if (this.eventObj.event_dates.includes(date)) {
        return true;
      }

      return false;
    }
  },
};
</script>

<style>
#card-container {
  margin-top: 20px;
}

.form-check-label {
  padding-right: 7px;
  padding-left: 2px;
}

.form-check-input {
  width: 13px;
  height: 13px;
  margin-top: 7px;
}

#chk-group-days {
  margin-top: 15px;
}

#card-body-container {
  padding-left: 0;
  padding-right: 0;
}

.btn {
  margin-top: 15px;
}
</style>