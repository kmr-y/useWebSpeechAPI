<template>
  <v-container fulid fill-height @click="screen()">
    <v-layout column fill-height>
      <v-flex xs2>
        <v-layout align-center justify-center fill-height row>
          <div class="headline">おはなしをするときはがめんをタッチしてね！</div>
        </v-layout>
      </v-flex>
      <v-flex xs10>
        <v-layout align-center justify-center fill-height>
          <img src="../assets/animal_fukurou.png">
        </v-layout>
        <v-tooltip bottom v-model="speech" v-show="speech">{{speech}}</v-tooltip>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import firebase from "firebase";
const synth = window.speechSynthesis;
export default {
  name: "HelloWorld",
  data() {
    return {
      text: "",
      speech: ""
    };
  },
  watch: {
    text(val) {
      if (val) {
        this.textToSpeech(val);
        this.updateHistory(val, "text");
      }
    }
  },
  created() {
    // speechSynthesis.speak() without user activation is no longer allowed since M71, around December 2018. See https://www.chromestatus.com/feature/5687444770914304 for more details
    // let msg = "話すときは画面をタッチしてね！";
    // this.textToSpeech(msg);
  },
  methods: {
    screen() {
      this.recognition();
    },
    recognition() {
      let self = this;
      var recognition = new webkitSpeechRecognition();
      recognition.lang = "ja-JP";
      recognition.start();
      recognition.onresult = function(event) {
        let speech = event.results[0][0].transcript;
        self.setSpeech(speech);
        self.updateHistory(speech, "speech");
      };
    },
    setSpeech(speech) {
      this.speech = speech;
      firebase
        .database()
        .ref("speech")
        .set(speech);
    },
    onText() {
      let self = this;
      firebase
        .database()
        .ref("text")
        .on("value", function(snapshot) {
          let text = snapshot.val();
          self.text = text;
        });
    },
    textToSpeech(text) {
      let utterThis = new SpeechSynthesisUtterance(text);
      synth.speak(utterThis);
      utterThis.onerror = function(event) {
        console.log("SpeechSynthesisUtterance Error: " + event.utterance.text);
      };
    },
    updateHistory(val, type) {
      let database = firebase.database().ref();
      let updates = {};
      let postData = {
        timeStamp: new Date().getTime(),
        type: type,
        value: val
      };
      var newPostKey = database.child("history").push().key;
      updates[`/history/${newPostKey}`] = postData;
      database.update(updates);
    }
  },
  mounted() {
    this.onText();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
img {
  max-width: 100%;
  max-height: 100%;
  width: auto;
  height: auto;
}
</style>
