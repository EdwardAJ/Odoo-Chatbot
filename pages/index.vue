<template>
  <div>
    <Chat
     @newOwnMessage="onNewOwnMessage" 
     :title="'Pudak Scientific Customer Service'"
     :initial-feed="feed"
     :new-message="message" />
     {{ state }}
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
        CHOOSEDEPARTMENT: 2
      },
      state: 1,
      keluhanState: 1,
      backendURL: "http://localhost:8006"
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
    handleIsiPertanyaan (message) {
      // Send message to the backend
      var url = this.backendURL + "/api/v1/pertanyaan/"
      axios.get({ pertanyaan: message}, url).then((response) => {
        console.log("response: ", response)
         // Reset to initial state...
        // this.resetState()
      }).catch(error => {
        console.log(error)
      })
    },
    handleIsiKeluhan (message) {
      // Choose the departments, call backend API
      this.getAllEmployee()
      // Change keluhanState
      this.keluhanState = this.keluhanStateEnum.CHOOSEDEPARTMENT
    },
    getAllEmployee () {
      console.log('GET ALL EMPLOYEE')
      axios.post(this.backendURL + "/api/v1/employee/").then((response) => {
        console.log(response.data);
      })
    },
    sendKeluhanToBackEnd () {

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
        } else if (this.keluhanState == this.keluhanStateEnum.CHOOSEDEPARTMENT) {
          if (this.validateMessageDepartment(message)) {
            this.sendKeluhanToBackEnd()
          }
        }
      }
    }
  }
}
</script>
<style lang = "scss">
    $primary: red;
    $window-height: 200px;
    @import "basic-vue-chat/src/assets/scss/main.scss";
</style>