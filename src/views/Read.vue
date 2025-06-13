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

    <div class="card-container">
      <div v-for="dbItem in db" :key="dbItem.name" class="card">
        <h3>名前:{{ dbItem.fields.name.stringValue }}</h3>
        <p>性別: {{ dbItem.fields.gender.stringValue }}</p>
      </div>
    </div>
  </div>

<!--編集画面遷移用-->
  <div class="Link">
  <RouterLink to="/Update">Update</RouterLink>
  </div>
</template>

<style>
.Link{
  text-align: center;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  background-color: #2b0d72;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  max-width: 200px;
}
</style>