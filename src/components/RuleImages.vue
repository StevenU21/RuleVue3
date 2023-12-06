<template>
  <div class="gallery-container">
    <h1>Imágenes de la Rule34</h1>
    <div class="header">
      <input type="text" v-model.trim="searchTerm" placeholder="Buscar..." class="search-input" />
      <button class="button" @click="searchPosts" :disabled="!searchTerm.trim()">Buscar</button>
    </div>

    <div v-if="loading" class="loading-message">
      <p>Cargando...</p>
    </div>

    <div v-if="error" class="error-message">
      <p>Error: {{ error }}</p>
    </div>

    <ul class="image-list">
      <li v-for="post in posts" :key="post.id" class="image-item">
        <img :src="post.file_url" alt="Rule34 Image" class="image" />
      </li>
    </ul>

    <div class="load-more">
      <button class="load-more-button" @click="loadMore" :disabled="loading">Cargar Más</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const posts = ref([]);
    const error = ref(null);
    const loading = ref(false);
    const page = ref(0);
    const searchTerm = ref('');

    const fetchData = async () => {
      loading.value = true;
      const limit = searchTerm.value ? 18 : 27;
      const apiUrl = `https://api.rule34.xxx/index.php?page=dapi&s=post&q=index&tags=${searchTerm.value}&limit=${limit}&pid=${page.value}&json=1`;

      try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        if (!response.ok || data.success === 'false') {
          throw new Error(data.message || `HTTP error! Status: ${response.status}`);
        }

        posts.value = searchTerm.value ? data : [...posts.value, ...data];
        error.value = null;
      } catch (error) {
        console.error("Error:", error.message || error);
        error.value = "Failed to fetch data. Please try again later.";
      } finally {
        loading.value = false;
      }
    };

    const searchPosts = () => {
      page.value = searchTerm.value ? 0 : page.value;
      fetchData();
    };

    const loadMore = () => {
      page.value += 1;
      fetchData();
    };

    onMounted(() => {
      fetchData();
    });

    return { posts, error, loading, searchTerm, searchPosts, loadMore };
  },
};
</script>
