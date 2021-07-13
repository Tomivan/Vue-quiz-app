<template>
<div id="quiz">

<div v-if="introStage">
<h1>Welcome to the quiz: {{title}} </h1>

<p> Some text here</p>

<button @click="startQuiz">Start!</button>
</div>

<div v-if="questionStage">
<question :question="questions[currentQuestion]"
          v-on:answer="handleAnswer"
          :question-number="currentQuestion+1"></question>
</div>
<div v-if="resultsStage">
You got {{correct}} right out {{questions.length}} questions. Your percentage is {{perc}}%
</div> 
</div>
</template>

<script>
import Vue from 'vue'

const quizData = 'https://api.myjson.com/bins/ahnlp';

Vue.component('question', {
    template: `
    <div>
    <strong>Question {{questionNumber}}:</strong>
    <strong>{{ question.text }} </strong>
    
    <div v-if="question.type === 'tf'">
    <input type=radio name="currentQuestion" id="trueAnswer" v-model="answer" value="t">
    <label for="trueAnswer">True</label><br/>
    <input type=radio name="currentQuestion" id="falseAnswer" v-model="answer" value="t">
    <label for="falseAnswer">False</label><br/>
    </div> 
    
    <div v-if="question.type === 'mc'">
    <div v-for="(mcanswer,index) in question.answers">
    <input type="radio" :id="'answer'+index" name="currentQuestion" v-model="answer" :value="mcanswer">
    <label :for="'answer'+index">{{mcanswer}}</label><br/>
    </div>
  </div>

  <button @click="submitAnswer">Answer</button>
  <div>`,
  data() {
      return {
          answer: ''
      }
  }, 
  props:['question', 'question-number'],
  methods:{
      submitAnswer:function() {
          this.$emit('answer', {answer:this.answer});
          this.answer = null;
      }
  }
});
const app = new Vue({
    el: '#quiz',
    data() {
        return {
            introStage:false,
            questionStage:false,
            resultsStage:false,
            title: ' ',
            questions: [],
            currentQuestion:0,
            answers:[],
            correct:0,
            perc: null
        }
    },
    created() {
        fetch(quizData)
          .then(res => res.json())
          .then(res => {
              this.title = res.title;
              this.questions = res.questions;
              this.introStage = true;
          })
    },
    methods: {
        startQuiz() {
            this.introStage = false;
            this.questionStage = true;
            console.log('test'+JSON.stringify(this.questions[this.currentQuestion]))
        },
        handleAnswer(e) {
            console.log('answer event ftw', e);
            this.answers[this.currentQuestion]=e.answer;
            if((this.currentQuestion+1) === this.questions.length) {
                this.handleResults();
                this.questionStage = false;
                this.resultsStage = true;
            } else {
                this.currentQuestion++;
            }
        },
        handleResults() {
            console.log('handle results');
            this.questions.forEach((a, index) => {
                if(this.answers[index] === a.answer) this.correct++;
            });
            this.perc = ((this.correct / this.questions.length)*100).toFixed(2);
            console.log(this.correct + '' +this.perc);
        }
    }
})
</script>