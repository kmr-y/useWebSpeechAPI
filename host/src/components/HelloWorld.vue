<template>
  <v-app id="inspire">
    <v-container>
      <v-layout column>
        <v-flex xs1>
          <v-text-field
            v-model="text"
            :append-icon="text ? 'send' : 'send'"
            box
            clear-icon="close"
            clearable
            label="Message"
            type="text"
            placeholder="Please input text."
            @click:append="setMessage"
            @keyup.enter="setMessage"
            @click:clear="clearMessage"
          ></v-text-field>
        </v-flex>
        <v-flex xs11>
          <v-layout row wrap>
            <div v-for="(item, key) in history" :key="key">
              <v-card dark color="accent" class="ma-1" v-show="item.type==='text'">
                <v-card-text>{{item.value}}</v-card-text>
              </v-card>
              <v-card dark color="primary" class="ma-1" v-show="item.type==='speech'">
                <v-card-text>{{item.value}}</v-card-text>
              </v-card>
            </div>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</template>

<script>
import firebase from "firebase";
export default {
  name: "HelloWorld",
  data() {
    return {
      text: "",
      history: Array
    };
  },
  methods: {
    clearMessage() {
      this.text = "";
    },
    setMessage() {
      firebase
        .database()
        .ref("text")
        .set(this.text);
      this.text = "";
    },
    onHistory() {
      let self = this;
      firebase
        .database()
        .ref("history")
        .on("value", function(snapshot) {
          let history = snapshot.val();
          self.reverseHistory(history);
        });
    },
    reverseHistory(history) {
      let tempHistory = [];
      Object.keys(history).forEach(function(key) {
        tempHistory.push(history[key]);
      });
      this.history = tempHistory.reverse();
    }
  },
  mounted() {
    this.onHistory();
  }
};
</script>

<style>
html {
  overflow: hidden;
}
</style>
