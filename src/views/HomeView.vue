<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

// リアクティブなデータ
const title = ref("うまのなまえ(Read)");
const db = ref([]);

// Firestore からデータを取得する関数
const getData = async () => {
  const project1 = "my-vue-app-56f5a"; // プロジェクトID
  const collection1 = "uma"; // コレクション名
  const url = `https://firestore.googleapis.com/v1/projects/${project1}/databases/(default)/documents/${collection1}`;

  try {
    const result = await axios.get(url);
    console.log(result.data);
    db.value = result.data.documents; // 取得したデータを db に格納
  } catch (error) {
    console.error("データ取得エラー:", error);
  }
};

// コンポーネントがマウントされたらデータを取得
onMounted(getData);
</script>

<template>
  <div>
    <h1>{{ title }}</h1>
    <ul>
      <li v-for="dbItem in db" :key="dbItem.name">
        {{ dbItem.fields.name.stringValue }}
      </li>
      <li v-for="dbItem in db" :key="dbItem.gender">
        {{ dbItem.fields.gender.stringValue }}
      </li>
    </ul>
  </div>
</template>