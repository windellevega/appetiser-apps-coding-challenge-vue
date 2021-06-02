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
          <div class="row">
            <div class="col-8">
              <h5 class="card-title">Calendar</h5>
            </div>
            <div class="col-4">
              <div aria-live="polite" aria-atomic="true" class="position-relative">
                <div class="toast-container position-absolute top-0 end-0 p-3">

                  <!-- Then put toasts within -->
                  <div class="toast text-white bg-success border-0" :class="toastVisible ? 'show' : ''" role="alert" aria-live="assertive" aria-atomic="true">
                    <div class="d-flex">
                      <div class="toast-body">
                        <i class="bi bi-check2"></i> Event successfully saved.
                      </div>
                      <button type="button" class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast" aria-label="Close"></button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <div class="container-fluid" id="card-body-container">
            <div class="row">
              <div class="col-3">
                <div class="alert alert-danger" role="alert" :class="validationErrors.length == 0 ? 'd-none' : ''">
                  <strong>The following errors were encountered: </strong><br/>
                  <span v-for="validationError in validationErrors" :key="validationError">
                    * {{ validationError }} <br/>
                  </span>
                </div>
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
                  <h3 class="calendar-header">
                    <a href="#" @click="getPreviousCalendarMonth"><span aria-hidden="true">&laquo;</span></a>
                    {{ currentMonthFormat + ' ' + currentYearFormat }}
                    <a href="#" @click="getNextCalendarMonth"><span aria-hidden="true">&raquo;</span></a>
                  </h3>
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
import { computed, onBeforeMount, reactive, ref } from 'vue';

export default {
  /* COMPOSITION API*/
  setup() {
    /* DATA */
    const eventDates = ref([]);
    const currentMonth = ref(4);
    const currentYear = ref(2021);
    const daysOfWeek = ref([
      { 'day': 'Mon', 'value': 1 },
      { 'day': 'Tue', 'value': 2 },
      { 'day': 'Wed', 'value': 3 },
      { 'day': 'Thu', 'value': 4 },
      { 'day': 'Fri', 'value': 5 },
      { 'day': 'Sat', 'value': 6 },
      { 'day': 'Sun', 'value': 0 },
    ]);
    const eventName = ref('');
    const eventFrom = ref('');
    const eventTo = ref('');
    const checkedDays = ref([]);
    const eventObj = ref({
      id: '',
      event_name: '',
      event_dates: []
    });
    const toastVisible = ref(false);
    const validationErrors = ref([]);

    /* METHODS */
    function getCurrentMonthYear() {
      var dt = new Date();
      currentMonth.value = dt.getMonth();
      currentYear.value = dt.getFullYear();
    }

    function getNextCalendarMonth() {
      currentYear.value = currentMonth.value == 11 ? currentYear.value + 1 : currentYear.value;
      currentMonth.value = currentMonth.value == 11 ? 0 : currentMonth.value + 1;
      populateCalendar();
    }

    function getPreviousCalendarMonth() {
      currentYear.value = currentMonth.value == 0 ? currentYear.value - 1 : currentYear.value;
      currentMonth.value = currentMonth.value == 0 ? 11 : currentMonth.value - 1;
      populateCalendar();
    }
    
    function populateCalendar() {
      eventDates.value = []

      // Get total days of the month
      const dt = new Date(currentYear.value, currentMonth.value + 1, 0);

      const daysInMonth = dt.getDate();
      
      // Push dates of month to eventDates
      for (let x = 1; x <= daysInMonth; x++) {
        const strDate = currentYear.value + "-" + (currentMonth.value + 1) + "-" + x;

        eventDates.value.push({
          dayOfWeek: x,
          date: moment(strDate).format('YYYY-MM-DD'),
          day: moment(strDate).format('ddd')
        });
      }
    }

    function saveEvent() {
      // axios.post('http://35.188.11.79/api/events', {
      axios.post('http://localhost:8000/api/events', {
        event_name: eventName.value,
        event_date_from: eventFrom.value,
        event_date_to: eventTo.value,
        event_days: checkedDays.value
      })
      .then((response) => {
        eventObj.value = response.data;
        showToast();
        validationErrors.value = [];
      })
      .catch((error) => {
        validationErrors.value = Object.values(error.response.data.errors).flat();
      });
    }

    function getEvent() {
      // axios.get('http://35.188.11.79/api/events')
      axios.get('http://localhost:8000/api/events')
      .then((response) => {
        eventObj.value = response.data;
      });
      
    }

    function dateIsInEventDates(date) {
      if (eventObj.value.event_dates.includes(date)) {
        return true;
      }

      return false;
    }

    function hideToast() {
      toastVisible.value = false;
    }

    function showToast() {
      toastVisible.value = true
      setTimeout(hideToast, 3000);
    }    

    /* COMPUTED */
    const currentMonthFormat = computed(() => {
      return moment(new Date(currentYear.value, currentMonth.value, 1)).format('MMM');
    });

    const currentYearFormat = computed(() => {
      return moment(new Date(currentYear.value, currentMonth.value, 1)).format('YYYY');
    });

    /* LIFECYCLE HOOKS */
    onBeforeMount(() => {
      getEvent(),
      getCurrentMonthYear(),
      populateCalendar()
    });

    return {
      eventDates,
      daysOfWeek,
      eventName,
      eventFrom,
      eventTo,
      checkedDays,
      eventObj,
      toastVisible,
      validationErrors,
      getNextCalendarMonth,
      getPreviousCalendarMonth,
      saveEvent,
      dateIsInEventDates,
      currentMonthFormat,
      currentYearFormat
    }
  }
}
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
  padding: 10px 0 0 0;
  border-top: 1px solid #9fa3a7;
  margin-top: 10px;
}

.toast {
  z-index: 5;
}

.btn {
  margin-top: 15px;
}

.calendar-header a {
  text-decoration: none;
  padding: 0 10px;
}
</style>