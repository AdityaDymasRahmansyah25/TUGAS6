<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">{{ form.id ? 'Update' : 'Save' }}</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div>
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
          <div class="button-group">
            <button @click="edit(article)" class="edit-button">Edit</button>
            <button @click="remove(article.id)" class="delete-button">Delete</button>
          </div>
        </div>
      </li>
    </ul>
    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
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
        const response = await axios[method](url, form);

        // Assuming successful save, update UI and reset form
        if (form.id) {
          // Update existing article
          const index = articles.value.findIndex(
            (article) => article.id === form.id
          );
          if (index !== -1) {
            articles.value.splice(index, 1, response.data);
          }
        } else {
          // Add new article
          articles.value.push(response.data);
        }
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function remove(articleId) {
      try {
        await axios.delete(`http://localhost:3000/articles/${articleId}`);
        articles.value = articles.value.filter(
          (article) => article.id !== articleId
        );
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    onMounted(load);

    return { form, articles, save, edit, remove };
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
}

.form {
  margin-bottom: 20px;
}

input[type="text"],
textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

ul {
  list-style: none;
  padding: 0;
}

.article-item {
  margin-bottom: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.article-item h3 {
  margin-top: 0;
}

.button-group {
  margin-top: 10px;
}

.edit-button {
  background-color: #007bff;
  color: #fff;
}

.delete-button {
  background-color: #dc3545;
  color: #fff;
}

.load-button {
  background-color: #28a745;
  color: #fff;
}
</style>
