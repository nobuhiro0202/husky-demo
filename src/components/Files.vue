<template>
  <div>
    <h1>画像一覧</h1>

    <ul>
      <li v-for="file in savedFiles" :key="file.name">
        <a @click="downloadImage(file)">{{ file.name }}</a>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      savedFiles: [], // 保存済みファイルの一覧を保持するためのリスト
    };
  },
  mounted() {
    // 画面がロードされたときに IndexedDB からデータを取得
    this.loadDataFromDB();
  },
  methods: {
    async loadDataFromDB() {
      // IndexedDB への接続
      const db = await this.openDB();

      // 保存済みファイルの一覧を取得
      const files = await this.getAllFilesFromDB(db);

      // IndexedDB との接続を閉じる
      db.close();

      // 取得したファイルをコンポーネントのデータにセット
      this.savedFiles = files;
    },
    openDB() {
      return new Promise((resolve, reject) => {
        const request = indexedDB.open('FileStorageDB', 1);

        request.onerror = (event) => {
          reject(new Error('IndexedDB のオープン中にエラーが発生しました。'));
        };

        request.onsuccess = (event) => {
          const db = event.target.result;
          resolve(db);
        };

        request.onupgradeneeded = (event) => {
          const db = event.target.result;
          db.createObjectStore('files', { keyPath: 'name' });
        };
      });
    },
    getAllFilesFromDB(db) {
      return new Promise((resolve, reject) => {
        const transaction = db.transaction(['files'], 'readonly');
        const objectStore = transaction.objectStore('files');
        const request = objectStore.getAll();

        request.onsuccess = (event) => {
          const files = event.target.result;
          resolve(files);
        };

        request.onerror = (event) => {
          reject(new Error('IndexedDB からのデータ取得中にエラーが発生しました。'));
        };
      });
    },
    downloadImage(file) {
      const blob = this.base64ToBlob(file.content);
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = file.name;
      link.click();
    },
    base64ToBlob(base64) {
      const binaryString = window.atob(base64);
      const arrayBuffer = new ArrayBuffer(binaryString.length);
      const uint8Array = new Uint8Array(arrayBuffer);

      for (let i = 0; i < binaryString.length; i++) {
        uint8Array[i] = binaryString.charCodeAt(i);
      }

      return new Blob([uint8Array], { type: 'application/octet-stream' });
    },
  },
};
</script>

<style scoped>
/* スタイルが必要な場合はここに追加 */
ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 10px;
  cursor: pointer;
}
</style>
