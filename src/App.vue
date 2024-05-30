<template>
  <div class="container">
    <!-- Form -->
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>

    <!-- List of Articles -->
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        {{ article.title }}<br />
        {{ article.content }}<br />
        <button @click="edit(article)">Edit</button>
        <button @click="deleteArticle(article.id)">Delete</button>
      </li>
    </ul>

    <!-- Load Button -->
    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, { title: form.title, content: form.content });

        // Assuming successful save, update UI and reset form
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        // Reset form
        form.id = '';
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        // Remove the article from the local array first
        articles.value = articles.value.filter((article) => article.id !== id);
        // Then send delete request to the server
        await axios.delete(`http://localhost:3000/articles/${id}`);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Lato:wght@400;700&display=swap');

body {
  background-color: #2c3e50;
  font-family: 'Lato', sans-serif;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #34495e;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  color: #ecf0f1;
}

.form {
  margin-bottom: 20px;
}

.form input[type="text"],
.form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #95a5a6;
  border-radius: 4px;
  box-sizing: border-box;
  background-color: #7f8c8d;
  color: #ecf0f1;
}

.form button {
  background-color: #2980b9;
  color: #ecf0f1;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form button:hover {
  background-color: #1f5f86;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #2c3e50;
  margin-bottom: 10px;
  padding: 15px;
  border: 1px solid #34495e;
  border-radius: 4px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article-item button {
  background-color: #e74c3c;
  color: #ecf0f1;
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5px;
}

.article-item button:hover {
  background-color: #c0392b;
}

.article-item button:nth-child(2) {
  background-color: #27ae60;
}

.article-item button:nth-child(2):hover {
  background-color: #1e8449;
}

.load-button {
  background-color: #3498db;
  color: #ecf0f1;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: block;
  width: 100%;
  text-align: center;
}

.load-button:hover {
  background-color: #2980b9;
}
</style>
