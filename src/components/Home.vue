<template>
  <v-container grid-list-xl>
    <v-layout text-xs-center wrap>
      <v-flex v-for="reading in readings" :key="reading.DeviceId" xs12 sm6>
        <v-card >
          <v-card-title primary-title>
            <h3 class="headline mb-0">Readings {{displayUnixDateTime(reading.DateTime)}}</h3>
          </v-card-title>
          <v-card-text style="text-align: left">
            <div>Temperature: {{truncateReading(reading.TempC)}}C ({{inFahrenheit(reading.TempC)}}F)</div>
            <div>Humidity: {{truncateReading(reading.humidity)}}%</div>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  /* eslint-disable no-console */
  import axios from 'axios';
  import dayjs from 'dayjs';

  const SMARTRRD_URL = 'http://gretel.kungfoo.info:8080';

  export default {
    data: () => ({
      readings: []
    }),
    mounted: function () {
      this.getReadings();
    },
    methods: {
      getReadings() {
        axios.get(SMARTRRD_URL + '/getReadings')
          .then((resp) => {
            this.readings = resp.data.data;
          })
          .catch((err) => {
            console.error(err.message);
          }).
          finally(() => {
            setTimeout(this.getReadings, 10 * 1000);
          });
      },
      displayUnixDateTime(unix) {
        let dt = dayjs.unix(unix);
        let now = dayjs();
        if (now.diff(dt, 'second') < 60) {
          return now.diff(dt, 'second') + 'sec ago';
        }
        if (now.diff(dt, 'minute') < 60) {
          return now.diff(dt, 'minute') + 'min ago';
        }
        if (now.diff(dt, 'hour') < 60) {
          return now.diff(dt, 'hour') + 'hr ago';
        }
        return 'on ' + dt.format('DD MMM, HH:mm:ss');
      },
      truncateReading(reading) {
        return Math.floor(reading * 10)/10;
      },
      inFahrenheit(celsius) {
        return this.truncateReading(32 + (celsius * 9/ 5));
      }
    }
  }
</script>

<style>

</style>