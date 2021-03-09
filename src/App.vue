<script>
import AltChat from '../src/components/AltChat.vue';
import AltTable from "@/components/AltTable";
export default {
  components: {AltTable}
}

</script>

<template>
  <!--
  Set your experiment id here and
  Define your data sources with the trials attribute
  -->
  <Experiment
      title = "Language Pragmatics Chat Experiment"
      id="174-language-pragmatics"

  >
    <!-- The contents of the #title template slot will be displayed in the upper left corner of the experiment -->
    <template #title>
      <div>Interactive chat experiment</div>
    </template>

    <!-- The contents of the #screens template slot define your experiment -->
    <template #screens>
      <Screen :title="'Welcome'">
        Welcome to our experiment.
        <br />
        <br />
        We are so happy you joined.
        <br />
        <br />
        Usually, nobody ever plays with us. But now you are here.
        <br />
        Yay! We will have so much fun together :)))
        <button @click="$magpie.nextScreen();">Begin the experiment</button>
      </Screen>

      <Screen :title="'General Instructions'">
        In this experiment, you will evaluate the fairness of a soccer club (Exampleton FC) during the last season.
        In the table below you can see summary statistics of different fairness-related events counted for multiple
        soccer clubs during the last season.
        <br />
        <br />
        <AltTable :content="train.content"></AltTable>
        <br />
        Note that you can sort the table in descending or ascending order by clicking on the descriptions in the
        head row.
        <br />
        <br />
        Your task will be to discuss the topic with a human chat partner, arguing either for or against Exampleton
        FC. During the discussion with your chat partner you will be able to inspect a table similar to the one
        above, and use its information as a basis for your arguments.
        <br />
        Before and after the Chat, you will be asked to rate Exampleton FC's fairness based on the summary statistics.
        <button
            @click="
            $magpie.nextScreen();
            $magpie.addResult({ question: 'condition', answer: (parseInt($magpie.socket.variant) - 1)});
          "
        >
          Continue
        </button>
      </Screen>

      <Screen :title="'Your Opinion'">
        <template #0="{responses}">
          <AltTable :content="trial.content[parseInt($magpie.socket.chain)]"></AltTable>
          <p>How fair did Exampleton FC play during the last season?</p>
          <SliderInput
              left="Very Unfair"
              right="Very Fair"
              initial="50"
              :response.sync="responses.slider"
          />
          <button
              @click="
              $magpie.addResult({question: 'pre', answer:  responses.slider});
              $magpie.nextScreen();
              setChatTimer();
          "
          >
            Continue
          </button>
        </template>
      </Screen>

      <ConnectInteractive />

      <Screen>
        <AltTable :content="trial.content[parseInt($magpie.socket.chain)]"></AltTable>
        <p>{{ trial.conditions[ (parseInt($magpie.socket.variant) - 1)] }}</p>
        <AltChat></AltChat>
        <button @click="leaveChat();">Leave Chat</button>
      </Screen>

      <Screen :title="'Your Opinion'">
        <template #0="{responses}">
          <AltTable :content="trial.content[parseInt($magpie.socket.chain)]"></AltTable>
          <p>How fair did Exampleton FC play during the last season?</p>
          <SliderInput
              left="Very Unfair"
              right="Very Fair"
              initial="50"
              :response.sync="responses.slider"
          />
          <button @click="$magpie.addResult({question: 'post', answer:  responses.slider}); $magpie.nextScreen();">Continue</button>
        </template>
      </Screen>

      <Screen key="additional-information" title="Additional information">
        <template #0="{ responses }">
          <p>
            Answering the following questions is optional, but your answers will
            help us analyze our results.
          </p>
          <div style="text-align: left; width: 200px; margin: 0 auto">
            <p>
              <label
              >Age
                <input v-model="responses.age" type="number" max="110" min="18"
                /></label>
            </p>
            <p>
              <label
              >Gender
                <select v-model="responses.gender">
                  <option value="male">male</option>
                  <option value="female">female</option>
                  <option value="other">other</option>
                </select></label
              >
            </p>
            <p>
              <label
              >Level of Eduction
                <select v-model="responses.education">
                  <option value="Graduated Highschool">
                    Graduated Highschool
                  </option>
                  <option value="Graduated College">Graduated College</option>
                  <option value="Higher degree">Higher degree</option>
                </select></label
              >
            </p>
            <p>
              <label
              >Native languages
                <input
                    v-model="responses.languages"
                    type="text"
                    placeholder="langauge(s) spoken at home when you were a child"
                    style="width: 400px"
                /></label>
            </p>
            Further comments
            <TextareaInput :response.sync="responses.comments"></TextareaInput>
          </div>

          <button
              @click="
              $magpie.addResult({question: 'age', answer:  responses.age});
              $magpie.addResult({question: 'gender', answer:  responses.gender});
              $magpie.addResult({question: 'education',answer: responses.education});
              $magpie.addResult({question: 'languages',answer: responses.languages});
              $magpie.addResult({question: 'comments',answer: responses.comments});
              $magpie.nextScreen();
            "
          >
            Next
          </button>
        </template>
      </Screen>

      <!--  <DebugResults /> -->
      <SubmitResults />

      <!-- While developing your experiment, using the DebugResults screen is fine,
      once you're going live, you can use the <SubmitResults> screen to automatically send your experimental data to the server. -->

      <Screen :title="'Thanks!'">
        Goodbye
      </Screen>
    </template>
  </Experiment>
</template>



<script>
// Load data from csv files as javascript arrays with objects
import _ from 'lodash';

import AltChat from '../src/components/AltChat.vue';
import AltTable from '../src/components/AltTable.vue';
import Vue from "vue";


// get our own random generators

const SEED = 23111995;


function RandomInt(seed) {
  this._seed = seed % 2147483647;
  if (this._seed <= 0) this._seed += 2147483646;
}

RandomInt.prototype.next = function () {
  this._seed = (this._seed * 16807) & 0xffffffff;
  return (this._seed - 1) | 0;
};

RandomInt.prototype.nextFloat = function () {
  // We know that result of next() will be 1 to 2147483646 (inclusive).
  return Math.abs((this.next() - 1) / 2147483646);
};

var rng = new RandomInt(SEED);

// generate the data

function generateData(head, rows, vmin, vmax) {
  vmin = (typeof vmin !== 'undefined') ?  vmin : 0;
  vmax = (typeof vmax !== 'undefined') ?  vmax : 20;
  var data = [];
  for (var i=0; i < rows.length; i ++){
    data.push([]);
    data[i][0] = rows[i];
    for (var j=1; j < head.length; j ++){
      data[i][j] = Math.floor(rng.nextFloat() * vmax) + vmin;
    }
  }
  return {head:head, data:data}
}

var INTROTABLE = generateData(["Name", "Fouls", "Red Cards", "Spit at people"], ["Exampleton FC", "VFL Osnabrück", "Hoffenham Hotspur"]);
var HEADS = ["Name", "Fouls caused", "Red Cards", "Injuries caused", "Injuries suffered", "Referees attacked", "starred Nutella commercials", "People died during aftermatch parties"];
var ROWS = ["Exampleton FC", "Grasshoppers Zürich", "FC Kafd", "RM Anova"];
var CONTENT = [generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS),
  generateData(HEADS, ROWS)];

console.log(CONTENT);

export default {
  name: "InteractiveExperiment",
  components: {
    AltChat,
    AltTable
  },
  data() {
    const train = {content: INTROTABLE};
    const trial = {
      conditions:["Convince your chat partner that Exampleton FC played unfair.",
        "Convince your chat partner that Exampleton FC played fair."],
      content: CONTENT
    };
    return {
      train,
      trial,
      chatTimer: null
    };
  },
  methods: {
    submitChat() {
      var container = [];
      for(var i=0; i<document.getElementById('chatbox_1').children.length; i++){
        var msg = document.getElementById('chatbox_1').children[i];
        var author = (msg.className === "message me") ?  "me" : "partner";
        var msg_text = msg.textContent;
        container.push([author, msg_text]);
      }
      this.$magpie.addResult({ question: "Chat messages", answer: container});
      console.log(this.$magpie.socket);
    },
    leaveChat() {
      var notice = document.getElementById('noticebox_1').firstChild.textContent;
      var partner_left = notice === "Your chat partner has left the chat. Please click [leave chat] to finish the experiment.";
      var time_passed = (new Date() - new Date(this.chatTimer)) > (1000 * 60 * 1); // TODO: set the timer to 15 minutes
      if (partner_left || time_passed) {
        this.submitChat();
        this.$magpie.nextScreen();
      } else {
        alert("Please discuss with your partner for at least 15 minutes before leaving.");
      }
    },
    setChatTimer() {
      this.chatTimer = new Date();
    }
  }
}
</script>
