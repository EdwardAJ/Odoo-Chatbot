<template>
  <div>
    <Chat 
     :new-message="message" />
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
      message: {
        id: 1,
 
        contents: 'hello',
        date: null
      }
    }
  }
}
</script>