<script>
import Files from './components/Files.vue';
export default {
  components: {
    Files,
  },
  data() {
    return {
      selectedFiles: [],
    };
  },
  methods: {
    handleFileChange(event) {
      const files = event.target.files;
      for (let i = 0; i < files.length; i++) {
        this.selectedFiles.push(files[i]);
      }
    },
    async uploadFiles() {
      if (this.selectedFiles.length > 0) {
        // IndexedDB への接続
        const db = await this.openDB();
        console.log(this.selectedFiles);

        // ファイルの内容を Base64 エンコードして IndexedDB に保存
        for (let i = 0; i < this.selectedFiles.length; i++) {
          const file = this.selectedFiles[i];
          const content = await this.readFileAsBase64(file);
          await this.saveToDB(db, file.name, content);
        }

        // IndexedDB との接続を閉じる
        db.close();

        console.log('アップロードが完了しました。');
      } else {
        console.warn('ファイルが選択されていません。');
      }
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
    readFileAsBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (event) => {
          resolve(event.target.result.split(',')[1]);
        };
        reader.onerror = (event) => {
          reject(new Error('ファイルの読み込み中にエラーが発生しました。'));
        };
        reader.readAsDataURL(file);
      });
    },
    saveToDB(db, fileName, content) {
      return new Promise((resolve, reject) => {
        const transaction = db.transaction(['files'], 'readwrite');
        const objectStore = transaction.objectStore('files');
        const request = objectStore.put({ name: fileName, content });

        request.onsuccess = (event) => {
          resolve();
        };

        request.onerror = (event) => {
          reject(new Error('IndexedDB への保存中にエラーが発生しました。'));
        };
      });
    },
  },
};
</script>

<template>
  <div>
    <h1>複数画像アップロード & IndexedDB 保存</h1>

    <input type="file" multiple @change="handleFileChange" />

    <button @click="uploadFiles">アップロード & 保存</button>
    <Files />
  </div>
</template>

<style scoped>
/* スタイルが必要な場合はここに追加 */
</style>
