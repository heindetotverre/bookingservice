<template>
  <h1>Reserveren padel</h1>
  <div>
    <h3>accounts</h3>
    <ul>
      <li v-for="person of people" :key="person">
        {{ person }}
      </li>
    </ul>
  </div>
  <div>
    <label for="1">IS TEST</label>
    <input type="checkbox" id="1" v-model="testValue" />
  </div>
  <div v-if="testValue">
    <label for="test-time">test moment for cron</label>
    <select
          name="time"
          id="test-time"
          v-model="testTime"
        >
          <option value="">--Please choose an option for the cron job to run during a test--</option>
          <option
            v-for="time of timeOptions()"
            :key="time"
            :selected="time === testTime"
            :value="time"
          >
            {{ time }}
          </option>
        </select>
  </div>
  <div class="m-t-1">
    <div>
      <div>
        <label for="date-select">Dag:</label>
        <input
          type="date"
          id="date-select"
          v-model="chosenDate"
        />
      </div>
      <div>
        <label for="time-select">Tijd:</label>
        <select
          name="time"
          id="time-select"
          v-model="defaultTime"
        >
          <option value="">--Please choose an option--</option>
          <option
            v-for="time of timeOptions()"
            :key="time"
            :selected="time === defaultTime"
            :value="time"
          >
            {{ time }}
          </option>
        </select>
      </div>
    </div>
    <h2>Direct reserveren om {{ presentationDate }} - {{ defaultTime }}:</h2>
    <div class="m-t-1">
      <button @click="reserve({ schedule: 'direct' })">Reserveren</button>
    </div>
  </div>
  <div class="m-t-1">
    <div class="inline">
      <h2>Gepland reserveren {{ presentationDate }} - {{ defaultTime }}</h2>
    </div>
    <div>
      <button @click="reserve({ schedule: 'set' })">Zet geplande reservering aan</button>
    </div>
    <div class="m-t-1">
      <button @click="reserve({ schedule: 'cancel' })">Annuleer geplande reservering</button>
    </div>
    <div class="m-t-1">
      <button @click="reserve({ schedule: 'check' })">Controleer geplande reservering</button>
    </div>
  </div>
  <div>
    <p v-if="isLoading">LOADING ....</p>
    <p class="red" v-if="!parsedResult">{{ bookingResult }}</p>
    <div v-else>
      <h2 v-for="line of parsedResult" :key="line">{{ line }}</h2>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import moment from 'moment';
import momentTimezone from 'moment-timezone'
import { onMounted, ref, computed } from 'vue'

  const bookingResult = ref(null)
  const chosenDate = ref('')
  const defaultTime = ref('19:00')
  const testValue = ref(false)
  const isLoading = ref(false)
  const testTime = ref('11:00')

  onMounted(() => {
    date()
  })

  const presentationDate = computed(() => {
    return moment(chosenDate.value).format('dddd DD-MM-YYYY')
  })

  const date = () => {
    const today = moment()
    today.add(3, 'days')
    chosenDate.value = today
  }

  const parsedResult = computed(() => {
    if (bookingResult.value === null) {
      return null
    }

    if (typeof bookingResult.value !== 'string' && Object.values(bookingResult.value).length) {
      return Object.values(bookingResult.value)
    }
    return null
  })

  const people = [
    'Jonathan Ouwehand',
    'Patrick Gieling',
    'Ricky de Haan',
    'Matthias Poortvliet' 
  ]

  const timeOptions = () => {
    const arr = []
    for (let i = 14; i < 46; i++) {
      const hour = Math.floor(i / 2)
      const minutes = i % 2 === 0 ? '00' : '30'
      arr.push(`${hour}:${minutes}`)
    }
    return arr
  }

  const reserve = async ({ schedule }) => {
    bookingResult.value = null

    let url

    if (schedule === 'direct') {
      url = `${process.env.VUE_APP_BOOKING_URL}`
    }
    if (schedule === 'set') {
      url = `${process.env.VUE_APP_BOOKING_URL}-start-scheduled-booking`
    }
    if (schedule === 'cancel') {
      url = `${process.env.VUE_APP_BOOKING_URL}-stop-scheduled-booking`
    }
    if (schedule === 'check') {
      url = `${process.env.VUE_APP_BOOKING_URL}-check-scheduled-booking`
    }

    const date = formatToRTFC(momentTimezone(chosenDate.value).tz('Europe/Amsterdam'))

    const testDateTime = createTestDateTimeMoment(testTime.value)

    const payload = {
      date: date,
      time: defaultTime.value,
      people: people,
      test: testValue.value,
      testDateTime: testDateTime
    }

    console.log('LOGGING PAYLOAD', payload)

    try {
      isLoading.value = true
      const { data } = await axios.post(url, payload)
      bookingResult.value = data
      isLoading.value = false
    } catch (error) {
      isLoading.value = false
    }
  }

  const createTestDateTimeMoment = (time) => {
    console.log('test time creation time before manipulation', time)

    const date = moment()
    const dateTimezoned = momentTimezone().tz('Europe/Amsterdam')
    const [hour, minutes] = time.split(':')
    date.set('hour', hour)
    date.set('minutes', minutes)

    const [hourtz, minutestz] = time.split(':')
    dateTimezoned.set('hour', hourtz)
    dateTimezoned.set('minutes', minutestz)
    console.log('test time creation time after manipulation TIMEZONED', `${dateTimezoned.hours()}:${dateTimezoned.minutes()}`)
    return formatToRTFC(dateTimezoned)
  }

  const formatToRTFC = (moment) => {
    const f = "ddd, DD MMM YYYY HH:mm:ss ZZ"
    return moment.format(f)
  }
</script>
<style scoped>
.m-t-1 {
  margin-top: 20px;
}

.m-t-2 {
  margin-top: 60px;
}

.inline {
  display: inline
}

.red {
  color: red;
}
</style>