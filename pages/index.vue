<template>
  <div>
    <Chat
     @newOwnMessage="onNewOwnMessage" 
     :title="'Pudak Scientific Customer Service'"
     :initial-feed="feed"
     :new-message="message" />
   </div>
</template>
<script>

import Chat from 'basic-vue-chat'
import axios from 'axios'

export default {
  components: {
    Chat
  },
  data(){
    return {
      message: {},
      pudakAuthorId: 1,
      feed: [
        {
          id: 1,
          author: null,
          contents: "Ketik 'bertanya' untuk bertanya, atau 'keluhan' untuk keluhan.",
          date: null
        }
      ],
      // State: 1 for initial, 2 for 'bertanya' and 3 for 'keluhan'
      stateEnum: {
        INITIAL: 1,
        PERTANYAAN: 2,
        KELUHAN: 3
      },
      // KeluhanState: 0 for handleIsiKeluhan, 1 for chooseDepartment.
      keluhanStateEnum: {
        ISIKELUHAN: 1,
        ISIEMAIL: 2,
        CHOOSEDEPARTMENT: 3
      },
      state: 1,
      keluhanState: 1,
      backendURL: "https://strong-monkey-91.localtunnel.me",
      keluhanContent: "",
      email: "",
      departments: {}
    }
  },
  methods: {
    resetKeluhanState () {
      this.keluhanState = this.keluhanStateEnum.ISIKELUHAN
    },
    resetState() {
      this.state = this.stateEnum.INITIAL
    },
    pushToFeed (element) {
      this.feed.push(element)
    },
    changeFeedElementDate () {
      this.feed[this.feed.length - 1].date = null
    },
    validateInitialMessage (message) {
      if (message == 'bertanya') {
        this.handleBertanyaRequest()
      } else if (message == 'keluhan') {
        this.handleKeluhanRequest()
      } else {
        this.sendMessage("Ketik 'bertanya' untuk bertanya, atau 'keluhan' untuk keluhan.")
      }
    },
    validateMessageDepartment (message) {
      return true
    },
    handleBertanyaRequest () {
      this.sendMessage('Silakan ketik pertanyaan Anda.')
      this.state = this.stateEnum.PERTANYAAN
    },
    handleKeluhanRequest () {
      this.sendMessage('Silakan ketik keluhan Anda.')
      this.state = this.stateEnum.KELUHAN
    },
    handlePertanyaanSuccess (response) {
      this.sendMessage(response.data.data)
      this.resetState()
      this.sendMessage("Ketik 'bertanya' untuk bertanya, atau 'keluhan' untuk keluhan.")
    },
    handleIsiPertanyaan (message) {
      // Send message to the backend
      var url = this.backendURL + "/api/v1/pertanyaan"
      axios.get(url, { params: {
        pertanyaan: message
      } }).then((response) => {
        if (response.status != 200 && response.status != 201) {
          this.sendMessage("Silakan coba pertanyaan yang lain.")
        } else {
          this.handlePertanyaanSuccess(response)
        }
      }).catch(error => {
        console.log(error)
        this.sendMessage("Terdapat kesalahan. Silakan ulangi kembali.")
      })
    },
    handleIsiKeluhan (message) {
      this.keluhan = message
      this.sendMessage("Silakan masukkan e-mail Anda.")
      // Change keluhanState
      this.keluhanState = this.keluhanStateEnum.ISIEMAIL
    },
    handleEmail (message) {
      this.email = message
      // Get all departments
      this.handleDepartmentsFromBackEnd()
      this.sendMessage("Harap tunggu...")
    },
    handleDepartmentsFromBackEnd () {
      let url = this.backendURL + "/api/v1/employee/"
      console.log(url)
      axios.get(url).then((response) => {
        response.data.forEach(employeeElement => {
          if (!this.departments[employeeElement.department]) {
            this.departments[employeeElement.department] = employeeElement.department
          }
        })
        this.sendExistingDepartmentsMessage()
      }).catch(error => {
        console.log(error)
        this.sendErrorMessage()
      })
    },
    sendExistingDepartmentsMessage () {
      let newMessage = "Silakan ketik salah satu dari departemen berikut: "
      const departments = Object.keys(this.departments)
      for (const dept of departments) {
        newMessage += dept + ", "
      }
      this.sendMessage(newMessage)
      this.keluhanState = this.keluhanStateEnum.CHOOSEDEPARTMENT
    },
    handleDepartment (message) {
      if (this.validateDepartment(message)) {
        // Send to backend
        this.sendKeluhanRequest(message)
      } else {
        this.sendMessage("Silakan ketik departemen yang benar!")
      }
    },
    validateDepartment (message) {
      // If key exists
      if (!this.departments[message]) {
        return false
      }
      return true
    },
    sendKeluhanRequest (dept) {
      let payload = {
        pelanggan: {
          email: this.email
        },
        department: dept,
        isi: this.keluhanContent
      }
      let url = this.backendURL + "/api/v1/keluhan/"
      axios.post(url, payload).then((response) => {
        this.notifyAcceptedKeluhan(response)
      }).catch(error => {
        console.log(error)
        this.sendErrorMessages()
      })
      this.sendMessage("Harap tunggu...")
    },
    sendErrorMessage () {
      this.sendMessage("Terdapat kesalahan pada server. Silakan ulangi kembali.")
      this.resetKeluhan()
    },
    notifyAcceptedKeluhan (response) {
      this.sendMessage("Keluhan Anda telah diterima oleh: " + response.data.data.penanggungjawab.email)
      this.sendMessage("Berikut ini adalah respon: " + response.data.data.jawaban)
      this.resetKeluhan()
    },
    resetKeluhan () {
      this.keluhanContent = ""
      this.email = ""
      this.departments = {}
      this.resetKeluhanState()
      this.resetState()
      this.sendMessage("Ketik 'bertanya' untuk bertanya, atau 'keluhan' untuk keluhan.")
    },
    sendMessage (message) {
      // Construct new message
      var newMessage = this.constructMessageObject(message)
      this.pushToFeed(newMessage)
    },
    constructMessageObject (message) {
      var newMessage = {
        id: this.pudakAuthorId,
        author: null,
        contents: message,
        date: null
      }
      return newMessage
    },
    onNewOwnMessage (message) {
      // Change the last element in the feed:
      this.changeFeedElementDate()
      // Check for the states:
      if (this.state == this.stateEnum.INITIAL) {
        this.validateInitialMessage(message)
      } else if (this.state == this.stateEnum.PERTANYAAN) {
        this.handleIsiPertanyaan(message)
      } else if (this.state == this.stateEnum.KELUHAN) {
        if (this.keluhanState == this.keluhanStateEnum.ISIKELUHAN) {
          this.handleIsiKeluhan(message)
        } else if (this.keluhanState == this.keluhanStateEnum.ISIEMAIL) {
          this.handleEmail(message)
        } else if (this.keluhanState == this.keluhanStateEnum.CHOOSEDEPARTMENT) {
          this.handleDepartment(message)
        }
      }
    }
  }
}
</script>
<style lang = "scss">
    $secondary: red;
    $message-max-width: 100px;
    @import "@/node_modules/basic-vue-chat/src/assets/scss/main.scss";
</style>