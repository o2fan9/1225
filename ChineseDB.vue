<script setup lang="ts">
import { inject ,reactive, watch } from 'vue'
import { collection, getDocs, getFirestore, where, query, updateDoc ,addDoc ,doc,arrayUnion } from 'firebase/firestore'
import app from '@/components/settings/FirebaseConfig.vue'

let units = [
  { title: '單元一 古代詩人的字', value: 1 },
  { title: '單元二 判斷季節', value: 2 }
]
const state = reactive({
  correctCount :0,
  incorrectCount :0,
  choice: { title: '單元一 古代詩人的字', value: 1 },
  answer: [''],
  answers: [[],[]],
  message: [''],
  exams: [{ question: '', option: [], answer: '', answers:[], type: '' }]
})
const appAccount = inject('account',{name:'未登入',email:'',password:'',id:""})
async function generateQuestions() {
  console.log(state.choice)
  state.exams = []
  const queryExam = query(examCollection, where('unit', '==', state.choice))
  const querySnapshot = await getDocs(queryExam)
  querySnapshot.forEach((doc) => {
    console.log(doc.data())
    state.answers.push([])
    state.exams.push({
      question: doc.data().question,
      option: doc.data().options,
      answer: doc.data().answer,
      answers: doc.data().answers,
      type: doc.data().type
    })
  })
}

const db = getFirestore(app)
const examCollection = collection(db, 'Chinese')
generateQuestions()
watch(() => state.choice, generateQuestions)

// const querySnapshot = await getDocs(queryExam);

// let exams:{question:string, answer:string}[]=[];
// querySnapshot.forEach((doc) => {
//   exams.push({question:doc.data().question, answer:doc.data().answer});
//   console.log('${doc.id} => ${doc.data()}');
// });

// let answer = "";
// const state = reactive({ message: "", currentQuestion: 0 });
// function generateQuestion() {
//   if (exams[state.currentQuestion].answer === answer) {
//     state.message = "答案正確";
//    if (state.currentQuestion + 1 < exams.length) {
//       state.currentQuestion++;
//     }
//   } else {
//     state.message = "答案錯誤";
//   }
// }
//generateQuestion();

async function checkAnswers() {
  state.message = [] // clear previous messages
  state.correctCount=0
  state.incorrectCount=0
  for (let i in state.exams) {
    if (state.exams[i].type == 'random') {
      if (state.answer[i] !== state.exams[i].answer) {
        state.message[i] = '不正確'
        state.incorrectCount++
      } else {
        state.message[i] = '正確'
        state.correctCount++
      }
    } else {
      // console.log("multiple")
      // console.log(state.exams[i].answer.length)
      // console.log(state.answers[i].length)

      if (state.exams[i].answer.length === state.answers[i].length) {
        let correct = 0
        for (let item of state.answers[i]) {
          if (state.exams[i].answer.includes(item)) {
            correct++
          }
        }
        // console.log(correct)
        if (correct == state.exams[i].answer.length) {
          state.message[i] = '答案正確'
          state.correctCount++
        } else {
          state.message[i] = '答案錯誤'
          state.incorrectCount++
        }
      } else {
        state.message[i] = '答案錯誤'
        state.incorrectCount++
        console.log("error")
      }
    
}if (state.exams[i].type === 'checkbox') {
  if (state.exams[i].answers.length === state.answers[i].length) {
    let correct = 0
    for (let item of state.answers[i]) {
      if (state.exams[i].answers.includes(item)) {
        correct++}
}
if (correct == state.exams[i].answers.length) {
state.message[i] = '答案正確'
state.correctCount++
} else {
  state.message[i] = '答案錯誤'
  state.incorrectCount++}
} else {
   state.message[i] = '答案錯誤'
   state.incorrectCount++}
}
  }
  await updateDoc(doc(db,"user",appAccount.id),{subject:arrayUnion("Chinese")})
  await updateDoc(doc(db,"user",appAccount.id),{unit:arrayUnion("Chinese"+state.choice)})
  await addDoc(collection(db,"user/"+appAccount.id+"/record"),
  {subject:"Chinese",
unit:state.choice,
correctCount: state.correctCount,
incorrectCount:state.incorrectCount,
date: new Date()})
}
</script>
<template>
  <v-container>
    <v-select label="請選擇" v-model="state.choice" :items="units"> </v-select>
    <div style="position: relative; width: 100%; height: 0; padding-top: 56.2500%;
 padding-bottom: 0; box-shadow: 0 2px 8px 0 rgba(63,69,81,0.16); margin-top: 1.6em; margin-bottom: 0.9em; overflow: hidden;
 border-radius: 8px; will-change: transform;">
  <iframe loading="lazy" style="position: absolute; width: 100%; height: 100%; top: 0; left: 0; border: none; padding: 0;margin: 0;"
    src="https:&#x2F;&#x2F;www.canva.com&#x2F;design&#x2F;DAF3yD7LsRI&#x2F;view?embed" allowfullscreen="allowfullscreen" allow="fullscreen">
  </iframe>
</div>
    <div v-for="(exam, index) in state.exams" :key="index">
      <v-text-field
        v-if="exam.type == 'blank'"
        v-model="state.answer[index]"
        :label="exam.question"
        :messages="state.message[index]"
      ></v-text-field>
      <p v-if="exam.type === 'random'">
  
        <v-text-field
          v-model="state.answer[index]"
          :label="exam.question"
          :messages="state.message[index]"
        ></v-text-field>

        <span v-for="option in exam.option" :key="option">
          
          {{ option }}
        </span>
        {{ state.message[index] }}

        </p>
      <div v-if="exam.type === 'checkbox'">
      <p>{{ exam.question }}</p>
      <v-row>
      <span v-for="options in exam.option" :key="options">
        <v-checkbox  v-model="state.answers[index]" :label="options" :value="options" ></v-checkbox>
      </span>
    </v-row>
        {{ state.message[index] }}
        
    </div>
    </div>
    <v-btn color="primary" @click="checkAnswers">檢查答案</v-btn>

  </v-container>
</template>