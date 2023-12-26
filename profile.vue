<script setup lang="ts">
import { inject, reactive, watch } from 'vue';
import { doc, getFirestore, updateDoc, deleteField, collection, query, where, limit, getDocs, orderBy } from 'firebase/firestore';
import app from '@/components/settings/FirebaseConfig.vue'
import { FirebaseError } from '@firebase/util';

const db = getFirestore(app);
const login = inject('account', { name: '未登入', email: '', id: '', loginCount: 0 })
const account = reactive({
  name: login.name,
  id: login.id,
  loginCount: login.loginCount,
  record: [{ unit: 0, correctCount: 0, incorrectCount: 0 }]
})

watch(login, () => {
  if (login.email !== "") {
    account.name = login.name,
    account.loginCount = login.loginCount,
    account.id = login.id
  } else {
    account.name = '未登入',
    account.loginCount = 0
  }
}, {
  immediate: true
})
const state = reactive({
  message: '',
})

async function handleClick() {
  if (login.id === '') {
    state.message = '尚未登入無法更新'
    return
  }
  try {
    state.message = '更新中...'
    const userRef = doc(db, "user", login.id);
    await updateDoc(userRef, {
      name: account.name
    });
    state.message = '更新成功'
  } catch (e) {
    if (e instanceof FirebaseError) {
      state.message = e.message
    } else {
      state.message = '更新失敗'
    }
  }
}

async function deleteDoc() {
  const userRef = doc(db, "user", login.id);
  await updateDoc(userRef, {
    loginCount: Number(0)
  });

}

async function queryRecord(str: string) {
  if (login.id === '') {
    return
  }
  try {
    const examCollection = collection(db, "user/" + login.id + '/record')
    const queryRecord = query(examCollection, where('subject', '==', str), orderBy('date', 'desc'), limit(1))
    const querySnapshot = await getDocs(queryRecord)
    querySnapshot.forEach((doc) => {
      account.record[0].unit = doc.data().unit,
      account.record[0].correctCount = doc.data().correctCount,
      account.record[0].incorrectCount = doc.data().incorrectCount
    })
  } catch (e) {
    if (e instanceof FirebaseError) {
      state.message = e.message
    } else {
      state.message = '失敗'
    }
  }
}

</script>
<template>
  <v-container>
    <v-text-field v-model=account.name label="姓名"></v-text-field>
    {{ state.message }}
    <v-btn class="mr-2" color="primary" @click="handleClick">更新</v-btn>
    <v-card class="mx-auto mt-5" variant="outlined">
      <v-card-item>
        <v-card-title>
          學習進度
        </v-card-title>
        <v-card-subtitle>
          距離你上次登入......
        </v-card-subtitle>
      </v-card-item>

      <v-card-text>
        <div class="text-body-1 mb-2">
          <v-btn class="mr-2" color="secondary" @click="queryRecord('Physics')">物理</v-btn>
          <v-btn class="mr-2" color="secondary" @click="queryRecord('Chemical')">化學</v-btn>
          <v-btn class="mr-2" color="secondary" @click="queryRecord('Biology')">生物</v-btn>
          <v-btn class="mr-2" color="secondary" @click="queryRecord('Geology')">地科</v-btn>
        </div>
        <div class="text-body-1">
          <div v-if="account.record[0].unit != 0">
            單元：{{ account.record[0].unit }}，答對題數：{{ account.record[0].correctCount }}，答錯題數：{{
              account.record[0].incorrectCount }}
          </div>
        </div>

      </v-card-text>
      <v-card-actions>
        <v-btn>
          登入次數：{{ account.loginCount }}
        </v-btn>
        <v-btn class="mr-2" color="secondary" @click="deleteDoc">清空</v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>