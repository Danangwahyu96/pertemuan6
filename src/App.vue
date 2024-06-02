<template>
  <div class="app-container">
    <form @submit.prevent="save" class="form-container">
      <input type="text" v-model="form.title" placeholder="Title" class="input-field"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea-field"></textarea><br />
      <button type="submit" class="btn-save">Save</button>
    </form>
    <ul class="articles-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="btn-edit">Edit</button>
          <button @click="remove(article.id)" class="btn-delete">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="btn-load">Load</button>
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
        console.error('error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          // Update the existing article
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add the new article
          articles.value.push(response.data);
        }

        // Reset the form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return {
      form,
      articles,
      save,
      edit,
      remove,
      load,
    };
  },
};
</script>

<style scoped>
.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form-container {
  margin-bottom: 20px;
  background-color: #f9f9f9;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.input-field, .textarea-field {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 3px;
  font-size: 16px;
}

.btn-save, .btn-edit, .btn-delete, .btn-load {
  background-color: #007BFF;
  color: #fff;
  border: none;
  padding: 10px 15px;
  margin: 5px;
  cursor: pointer;
  border-radius: 3px;
  font-size: 14px;
}

.btn-delete {
  background-color: #DC3545;
}

.btn-load {
  margin-top: 20px;
}

.articles-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #fff;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.article-content {
  max-width: 70%;
}

.article-actions {
  display: flex;
  gap: 10px;
}
</style>
