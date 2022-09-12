<template>
  <div class='home'>
    <img src="@/assets/harbourlogo.png" />
    <hr />
    <DropZone @drop.prevent='drop' @change='selectedFile'/>
    <!-- <span class='file-info'>File: {{ dropzoneFile.name }}</span> -->
   <hr />
    <span v-show="success" class="success"> Contract(s) sent with <b>success!</b> </span>
    <hr/>
    <div v-show="showDialog" class="eventdetails">
      <p><b>{{selectedEvent.title}} - {{ selectedEvent.start && selectedEvent.start.format('DD/MM/YYYY') }}</b></p>
      <button @click="downloadPDF()">PDF</button>
    </div>
    <hr />
    <div class='call'>
      <vue-cal
        :events='events'
        :time='false'
        active-view='month'
        :disable-views="['years', 'year']"
        events-on-month-view='long'
        :on-event-click="onEventClick"
      />
      <hr />
    </div>
  </div>
</template>

<script>
import VueCal from 'vue-cal'
import 'vue-cal/dist/vuecal.css'
import DropZone from '@/components/DropZone.vue'
import { ref } from 'vue'

export default {
  name: 'HomeView',
  components: {
    VueCal,
    DropZone
  },
  data () {
    return {
      dropzoneFile: ref(['']),
      events: [],
      success: false,
      showDialog: false,
      selectedEvent: {}
    }
  },
  methods: {
    downloadPDF () {
      window.open(
        this.selectedEvent.contentFull,
        '_blank' // <- This is what makes it open in a new window.
      )
    },

    onEventClick (event, e) {
      this.selectedEvent = event
      this.showDialog = true

      // Prevent navigating to narrower view (default vue-cal behavior).
      e.stopPropagation()
    },

    drop: function (e) {
      this.dropzoneFile = ref([''])
      this.dropzoneFile = e.dataTransfer.files

      const data = new FormData()

      for (const file of this.dropzoneFile) {
        data.append('file[]', file)
      }

      const sendFile = async () => {
        const response = await fetch('http://127.0.0.1:5000/', {
          method: 'POST',
          body: data,
          headers: {
          }
        })
        const myJson = await response.json()
        console.log(myJson)

        for (const agreement of myJson) {
          console.log(agreement.data)
          const date = new Date(agreement.data)
          const year = date.toLocaleString('default', { year: 'numeric' })
          const month = date.toLocaleString('default', { month: '2-digit' })
          const day = date.toLocaleString('default', { day: '2-digit' })

          const formattedDate = day + '-' + month + '-' + year

          const event = { start: formattedDate, end: formattedDate, title: agreement.filename, contentFull: 'http://127.0.0.1:8887/' + agreement.filename }
          this.events.push(event)
          console.log(this.events)
        }
      }
      sendFile()
      setTimeout(() => {
        this.success = false
      }, 5000)
    },
    selectedFile: function () {
      this.dropzoneFile = document.querySelector('.dropzoneFile').files
      console.log(this.dropzoneFile)

      const data = new FormData()

      for (const file of this.dropzoneFile) {
        data.append('file[]', file)
      }

      const sendFile = async () => {
        const response = await fetch('http://127.0.0.1:5000/', {
          method: 'POST',
          body: data,
          headers: {
          }
        })
        const myJson = await response.json()

        for (const agreement of myJson) {
          console.log(agreement.filename)
          const date = new Date(agreement.data)
          const year = date.toLocaleString('default', { year: 'numeric' })
          const month = date.toLocaleString('default', { month: '2-digit' })
          const day = date.toLocaleString('default', { day: '2-digit' })

          const formattedDate = day + '-' + month + '-' + year

          const event = { start: formattedDate, end: formattedDate, title: agreement.filename, contentFull: 'http://127.0.0.1:8887/' + agreement.filename }
          this.events.push(event)
        }

        console.log(this.events)
        this.success = true
      }
      sendFile()
      setTimeout(() => {
        this.success = false
      }, 5000)
    }

  }
}

</script>

<style>
vue-cal {
  height: 600px;
  padding: 2px;
}

.home {
   height: 100vh;
   display: flex;
   flex-direction: column;
   justify-content: center;
   align-items: center;
   background-color: #f1f1f1;
}

.vuecal__event {
  background-color: lightblue;
  padding: 5px;
}

.call {
  height: 500px;
  width: 800px;
}

.success {
  color: #3c763d;
  background-color: #dff0d8;
  border-color: #d6e9c6;
  padding: 15px;
  border: 1px  solid transparent;

}

.eventdetails {
  background-color: #41b883;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 1px  solid transparent;
  width: 30%;
  padding: 15px;
}

</style>
