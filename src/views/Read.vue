<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

const title = ref("うまのなまえ(Read)");
const db = ref([]);
const message = ref("");
const editingId = ref(""); // 編集中のカードのid（Firestoreのドキュメント名の末尾）
const editData = ref({
  name: "",
  gender: "",
});

// Firestore からデータを取得
const getData = async () => {
  const project1 = "my-vue-app-56f5a";
  const collection1 = "uma";
  const url = `https://firestore.googleapis.com/v1/projects/${project1}/databases/(default)/documents/${collection1}`;
  try {
    const result = await axios.get(url);
    db.value = result.data.documents || [];
  } catch (error) {
    message.value = "データ取得エラー";
    console.error(error);
  }
};

// Firestore から該当ドキュメントを削除
const deleteData = async (docName) => {
  if (!window.confirm("本当に削除しますか？")) return;
  try {
    const url = `https://firestore.googleapis.com/v1/${docName}`;
    await axios.delete(url);
    await getData();
    message.value = "削除しました";
    setTimeout(() => (message.value = ""), 1200);
  } catch (error) {
    message.value = "削除エラー";
    console.error(error);
  }
};

// 編集ボタンを押したとき
const startEdit = (item) => {
  editingId.value = item.name.split("/").pop(); // ドキュメントIDだけを取得
  editData.value = {
    name: item.fields.name.stringValue,
    gender: item.fields.gender.stringValue,
  };
};

// 編集をキャンセル
const cancelEdit = () => {
  editingId.value = "";
  editData.value = { name: "", gender: "" };
};

// 編集内容を保存
const updateData = async (item) => {
  const project1 = "my-vue-app-56f5a";
  const collection1 = "uma";
  const docId = item.name.split("/").pop();
  // ここでクエリパラメータを手で追記
  const url = `https://firestore.googleapis.com/v1/projects/${project1}/databases/(default)/documents/${collection1}/${docId}?updateMask.fieldPaths=name&updateMask.fieldPaths=gender`;
  try {
    const updatedFields = {
      fields: {
        name: { stringValue: editData.value.name },
        gender: { stringValue: editData.value.gender },
      },
    };
    await axios.patch(url, updatedFields); // ←ここでparamsは使わない
    message.value = "更新しました";
    editingId.value = "";
    await getData();
    setTimeout(() => (message.value = ""), 1200);
  } catch (error) {
    message.value = "更新エラー";
    console.error("Firestoreエラー詳細:", JSON.stringify(error.response?.data, null, 2));
  }
};




onMounted(getData);
</script>

<template>
  <div>
    <h1>{{ title }}</h1>

    <div v-if="message" class="message">{{ message }}</div>
    <div v-if="db.length === 0" class="empty">データがありません</div>

    <div class="card-container">
      <div v-for="dbItem in db" :key="dbItem.name" class="card">
        <div class="card-header">
          <span class="icon">🐎</span>
          <span class="card-title">
            <!-- 編集中ならinput、それ以外は通常表示 -->
            <template v-if="editingId === dbItem.name.split('/').pop()">
              <input v-model="editData.name" />
            </template>
            <template v-else>
              {{ dbItem.fields.name?.stringValue }}
            </template>
          </span>
          <!-- 削除・編集ボタン -->
          <button class="delete-btn" @click="deleteData(dbItem.name)" title="削除">✕</button>
          <button v-if="editingId !== dbItem.name.split('/').pop()" class="edit-btn" @click="startEdit(dbItem)" title="編集">✎</button>
        </div>
        <div class="card-detail">
          <span>性別: </span>
          <!-- 編集中ならinput -->
          <template v-if="editingId === dbItem.name.split('/').pop()">
            <input v-model="editData.gender" />
          </template>
          <template v-else>
            {{ dbItem.fields.gender?.stringValue || '不明' }}
          </template>
        </div>
        <!-- 編集中なら保存/キャンセルボタン -->
        <div v-if="editingId === dbItem.name.split('/').pop()" class="edit-action">
          <button class="save-btn" @click="updateData(dbItem)">保存</button>
          <button class="cancel-btn" @click="cancelEdit">キャンセル</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.message {
  color: #e53935;
  text-align: center;
  margin: 16px 0;
}

.empty {
  text-align: center;
  color: #bbb;
  margin: 24px 0;
}

.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
  justify-content: center;
  margin: 20px 0 8px 0;
}
.card {
  border-radius: 12px;
  background: #fff;
  border: 1.5px solid #dadada;
  box-shadow: 0 3px 18px rgba(60,0,140,0.08);
  min-width: 190px;
  max-width: 220px;
  padding: 18px 16px 12px 16px;
  display: flex;
  flex-direction: column;
  gap: 11px;
  position: relative;
  transition: box-shadow 0.2s;
}
.card:hover {
  box-shadow: 0 6px 22px rgba(50,0,140,0.18);
  border-color: #a7a6fa;
}
.card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 2px;
}
.icon {
  font-size: 1.7em;
  background: #a7a6fa25;
  padding: 6px 10px;
  border-radius: 50%;
  margin-right: 5px;
}
.card-title {
  font-weight: bold;
  font-size: 1.12em;
  color: #3c0d72;
  flex: 1;
}
.delete-btn,
.edit-btn {
  background: none;
  border: none;
  color: #e53935;
  font-size: 1.19em;
  cursor: pointer;
  margin-left: 7px;
  border-radius: 50%;
  transition: background 0.16s;
}
.edit-btn {
  color: #2b0d72;
  font-size: 1.14em;
}
.delete-btn:hover {
  background: #ffeaea;
}
.edit-btn:hover {
  background: #eaeaff;
}
.card-detail {
  font-size: 1em;
  color: #444;
  padding-left: 8px;
}
.edit-action {
  margin-top: 6px;
  display: flex;
  gap: 12px;
  justify-content: flex-end;
}
.save-btn,
.cancel-btn {
  padding: 3px 13px;
  border-radius: 6px;
  font-size: 0.98em;
  border: none;
  cursor: pointer;
}
.save-btn {
  background: #6ecb63;
  color: #fff;
}
.cancel-btn {
  background: #ccc;
  color: #fff;
}
.save-btn:hover {
  background: #3cb82a;
}
.cancel-btn:hover {
  background: #aaa;
}
input {
  font-size: 1em;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 2px 8px;
  margin-left: 2px;
}
</style>

