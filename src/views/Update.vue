<script setup>
import axios from "axios";
import { ref } from "vue";

// 編集用データ（id, name, gender）
const editData = ref({
  id: "",         // 画面遷移時にセット
  name: "",
  gender: "",
});

const message = ref("");
const loading = ref(false);

// FirestoreのREST APIで更新
const updateData = async () => {
  message.value = "";
  loading.value = true;

  // 入力チェック
  if (!editData.value.id || !editData.value.name || !editData.value.gender) {
    message.value = "全項目を入力してください";
    loading.value = false;
    return;
  }

  const project1 = "my-vue-app-56f5a";
  const collection1 = "uma";
  const url = `https://firestore.googleapis.com/v1/projects/${project1}/databases/(default)/documents/${collection1}/${editData.value.id}`;

  try {
    const updatedFields = {
      fields: {
        name: { stringValue: editData.value.name },
        gender: { stringValue: editData.value.gender },
      }
    };

    // PATCH + updateMask で安全に更新
    const params = {
      updateMask: {
        fieldPaths: ["name", "gender"]
      }
    };
    const result = await axios.patch(url, updatedFields, { params });
    message.value = "更新が完了しました！";
    console.log("更新成功:", result.data);
  } catch (error) {
    message.value = "更新エラー：" + error.message;
    console.error("更新エラー:", error);
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <div>
    <h1>編集画面</h1>
    <form @submit.prevent="updateData" class="edit-form">
      <label for="name">名前:</label>
      <input type="text" id="name" v-model="editData.name" required />

      <label for="gender">性別:</label>
      <input type="text" id="gender" v-model="editData.gender" required />

      <button type="submit" :disabled="loading">{{ loading ? "更新中..." : "更新" }}</button>
    </form>
    <div v-if="message" class="message">{{ message }}</div>
  </div>
</template>

<style scoped>
.edit-form {
  max-width: 360px;
  margin: 24px auto;
  display: flex;
  flex-direction: column;
  gap: 14px;
}
label {
  font-weight: bold;
  margin-bottom: 2px;
}
input[type="text"] {
  padding: 8px 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
}
button {
  padding: 10px 0;
  border-radius: 7px;
  background: linear-gradient(90deg, #ff9966 0%, #ff5e62 100%);
  color: #fff;
  font-size: 1em;
  font-weight: bold;
  border: none;
  margin-top: 14px;
  cursor: pointer;
}
button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
.message {
  margin-top: 18px;
  color: #e53935;
  text-align: center;
}
</style>
