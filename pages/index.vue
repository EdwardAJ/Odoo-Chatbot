<template>
  <div>
    <Chat
     @newOwnMessage="onNewOwnMessage" 
     :title="'Pudak Scientific Customer Service'"
     :initial-feed="feed"
     :new-message="message" />
     {{ message }}
   </div>
</template>

<script>
import Chat from 'basic-vue-chat'
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
      // State: 0 for initial, 1 for 'bertanya' and 2 for 'keluhan'
      state: 0,
      // KeluhanState: 0 for handleIsiKeluhan, 1 for chooseDepartment.
      keluhanState: 0
    }
  },
  methods: {
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
    handleBertanyaRequest () {
      this.sendMessage('Silakan ketik pertanyaan Anda.')
      this.state = 1
    },
    handleKeluhanRequest () {
      this.sendMessage('Silakan ketik keluhan Anda.')
      this.state = 2
    },
    handleIsiPertanyaan (message) {
      // Send message to the backend
      this.state = 0
    },
    handleIsiKeluhan (message) {
      // Choose the departments, call backend API
      this.getAllEmployee()
      // Change keluhanState
      this.keluhanState = 1

    },
    getAllEmployee () {
      axios.post("http://localhost:8006/api/v1/employee/").then((response) => {
        console.log(response.data);
      })
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
    sendBertanyaRequest (message) {
      // Construct new JSON
      var request = {
        "pertanyaan" : message
      }
    },
    onNewOwnMessage (message) {
      // Change the last element in the feed:
      this.changeFeedElementDate()
      // Check for the states:
      if (this.state == 0) {
        this.validateInitialMessage(message)
      } else if (this.state == 1) {
        this.handleIsiPertanyaan(message)
      } else if (this.state == 2) {
        if (this.keluhanState == 0) {
            this.handleIsiKeluhan(message)
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