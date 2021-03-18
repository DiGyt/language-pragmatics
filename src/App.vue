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
        Thanks for participating in this experiment our experiment.
        <br />
        This experiment will take about 15 minutes.
        <br />
        <br />
        <button @click="$magpie.nextScreen();">start the experiment</button>
      </Screen>

      <Screen :title="'Instructions'">
        In this experiment, you will evaluate the fairness of a fictional soccer club (Exampleton FC) during the last season.
        In the table below you can see different fairness-related events counted for various football clubs over the last season.
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
        above, and use its information as a basis for your arguments. Please write full sentences to make your arguments
        in the discussion.
        Before and after the discussion, you will be asked to rate Exampleton FC's fairness based on their scores in the table.
        <br />
        <br />
        Before starting the actual experiment, you will have the opportunity to freely chat with your chat partner.
        Press [Continue] to match up with a chat partner.
        <br />
        <button
            @click="
            $magpie.nextScreen();
            setChatTimer();
          "
        >
          Continue
        </button>
      </Screen>

      <ConnectInteractive />

      <Screen :title="'Free Chat'">
        <p>In this chat, you can get to know your chat partner.<br />Press [continue experiment] if you are ready to start the experiment.</p>
        <AltChat :instance="'0'"></AltChat>
        <!-- TODO: set timer to reasonable value -->
        <button @click="leaveChat(0.2, 'Informal Chat');">continue experiment</button>
      </Screen>

      <Screen :title="'Pre-Test Evaluation'">
        <template #0="{responses}">
          <AltTable :content="trial.content[$magpie.socket.chain % 6]"></AltTable>
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
          "
          >
            Continue
          </button>
        </template>
      </Screen>

      <Screen :title="'Discussion notice'">
        <p>You will now enter the discussion part of our experiment. Please use full sentences to
          make an argument and please stick to the discussion topic during the chat.
        </p>
        <button
            @click="
            $magpie.nextScreen();
            setChatTimer();
        "
        >
          Enter Discussion
        </button>
      </Screen>

      <Screen :title="'Discussion'">
        <AltTable :content="trial.content[$magpie.socket.chain % 6]"></AltTable>
        <p>{{ trial.conditions[getCondition()] }}</p>
        <AltChat :instance="'1'"></AltChat>
        <!-- TODO: set timer to reasonable value -->
        <button @click="leaveChat(0.3, 'Trial Chat');">Leave Chat</button>
      </Screen>

      <Screen :title="'Post-Test Evalution'">
        <template #0="{responses}">
          <AltTable :content="trial.content[$magpie.socket.chain % 6]"></AltTable>
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
              $magpie.addResult({question: 'education', answer: responses.education});
              $magpie.addResult({question: 'languages', answer: responses.languages});
              $magpie.addResult({question: 'comments', answer: responses.comments});
              $magpie.addResult({question: 'experiment chain', answer: $magpie.socket.chain});
              $magpie.addResult({question: 'participant variant', answer: $magpie.socket.variant});
              $magpie.addResult({ question: 'condition', answer: trial.conditions[getCondition()]});
              $magpie.addResult({question: 'trial head', answer: trial.content[$magpie.socket.chain % 6].head});
              $magpie.addResult({question: 'trial data', answer: trial.content[$magpie.socket.chain % 6].data});
              $magpie.addResult({question: 'fairplay score', answer: trial.content[$magpie.socket.chain % 6].score});
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

const SEED = 464787977362;


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

var INTROTABLE = generateData(["", "Injuries caused", "Red Cards", "Spit at people"], ["Exampleton FC", "Christels Palace FC", "Hoffenham Hotspur"], 0, 10);
var HEADS = ["", "Injuries caused", "Hand play fouls", "Offside caused", "Tactical fouls", "Verbal Insults", "Arguments with referees"];
var CONDS = ["Convince your chat partner that Exampleton FC played unfair.",
  "Convince your chat partner that Exampleton FC played fair."];
var CONTENT = [ {head:HEADS, data: [    ["Exampleton FC", 8, 11,  6,  2, 3, 10],
    ["Hoffenham Hotspur", 7, 10,  5, 13, 11,  9],
    ["Red Bull Suessburg", 13,  8, 11, 11,  5,  8],
    ["Hertha B.Sc.", 7, 13, 11, 13, 14, 13],
    ["RM Anova", 10, 11, 12,  6,  5,  6]], score:"mostly fair"},
  {head:HEADS, data: [       ["Exampleton FC", 3, 11,  9,  6,  4,  9],
      ["Hoffenham Hotspur", 9, 12,  8, 14,  6, 11],
      ["Red Bull Suessburg", 5, 10, 10,  9, 13, 7],
      ["Hertha B.Sc.", 10,  5, 11, 14,  5, 12],
      ["RM Anova", 7,  9, 10,  5, 11, 10]], score:"mostly fair"},
  {head:HEADS, data: [       ["Exampleton FC", 10, 10,  7,  8,  7, 14],
      ["Hoffenham Hotspur", 6,  6,  6, 11,  5, 14],
      ["Red Bull Suessburg", 14, 13, 11,  9,  7, 12],
      ["Hertha B.Sc.",  7,  6,  6, 12, 13,  8],
      ["RM Anova", 10, 14,  5, 10, 12, 13]], score:"medium"},
  {head:HEADS, data: [       ["Exampleton FC", 13, 12,  7, 13,  8,  6],
      ["Hoffenham Hotspur", 12, 10,  7, 14,  9, 14],
      ["Red Bull Suessburg", 12,  5,  6, 13,  6,  5],
      ["Hertha B.Sc.",  6, 14,  6,  6,  9,  9],
      ["RM Anova", 14,  9, 14,  7, 12, 10]], score:"medium"},
  {head:HEADS, data: [       ["Exampleton FC", 18, 11,  8,  7, 10,  17],
      ["Hoffenham Hotspur",12, 14,  6, 11, 11, 13],
      ["Red Bull Suessburg", 5,  6, 10, 5, 11, 14],
      ["Hertha B.Sc.",  7,  7, 11, 12, 10,  6],
      ["RM Anova", 13,  6,  9,  9, 12, 11]], score:"mostly unfair"},
  {head:HEADS, data: [       ["Exampleton FC", 6, 15,  8, 9, 16,  11],
      ["Hoffenham Hotspur",11, 10, 13,  5,  8, 12],
      ["Red Bull Suessburg", 13,  8,  10,  8,  9, 11],
      ["Hertha B.Sc.",  5, 12, 13,  8,  5,  9],
      ["RM Anova", 5, 10,  7,  9, 14, 11]], score:"mostly unfair"}];


console.log(CONTENT);

export default {
  name: "InteractiveExperiment",
  components: {
    AltChat,
    AltTable
  },
  data() {
    const train = {content: INTROTABLE};
    const trial = {content: CONTENT, conditions: CONDS};
    return {
      train,
      trial,
      chatTimer: null
    };
  },
  methods: {
    submitChat(result_name) {
      var container = [];
      for(var i=0; i<document.getElementById('chatbox_1').children.length; i++){
        var msg = document.getElementById('chatbox_1').children[i];
        console.log(msg)
        var author = (msg.className === "message me") ?  "me" : "partner";
        var msg_text = msg.textContent;
        var time = msg.attributes.time.value;
        container.push([ "\n" + author + "\t" + time + "\t" + msg_text + "\n"]);
      }
      this.$magpie.addResult({ question: result_name, answer: container});
      console.log(this.$magpie.socket);
    },
    leaveChat(min_time, result_name) {
      var notice = document.getElementById('noticebox_1').firstChild.textContent;
      var partner_left = notice === "Your chat partner has left the chat. Please click [leave chat] to continue with the experiment.";
      var time_passed = (new Date() - new Date(this.chatTimer)) > (1000 * 60 * min_time); // TODO: set the timer to 15 minutes
      if (partner_left || time_passed) {
        this.submitChat(result_name);
        this.chatTimer = null;
        this.$magpie.nextScreen();
      } else {
        alert("Please chat with your partner for at least " + min_time.toString() + " minutes before continuing.");
      }
    },
    setChatTimer() {
      this.chatTimer = new Date();
    },
    getCondition(){
      var rand = Math.round(rng.nextFloat() + 1); // get an integer in [1, 2]
      return ((rand === this.$magpie.socket.variant) ?  0 : 1);
    }
  }
}
</script>
