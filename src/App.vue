<template>
  <div class="container">
    <div class="left">
      <form @submit.prevent="submitForm">
        <label for="title">Tytuł:
          <input type="text" id="title" v-model="formData.Title" required>
        </label>
        <button type="submit">Wyślij</button>
      </form>
    </div>

    <div class="right">
      <div v-if="fetchedData.length > 0">
        <NestedList :items="fetchedData" :button-index="0" :value="sortDirection" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import NestedList from './components/NestedList.vue';

export default {
  name: 'App',
  components: {
    NestedList,
  },
  data() {
    return {
      fetchedData: [],
      formData: {
        Title: '',
        ParentItemId: null,
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      axios.get('https://localhost:44313/api/item')
        .then((response) => {
          this.fetchedData = response.data;
          console.log(this.fetchedData);
        })
        .catch((error) => {
          if (error.response && error.response.status === 404) {
            console.log('Nie znaleziono danych.');
          }
        });
    },
    submitForm() {
      const data = {
        Title: this.formData.Title,
        ParentItemId: this.formData.ParentItemId !== '' ? this.formData.ParentItemId : null,
      };

      axios.post('https://localhost:44313/api/item', data)
        .then((response) => {
          console.log(response.data);
          this.fetchData();
          window.location.reload();
        })
        .catch((error) => {
          console.error(error);
        });

      this.formData.Title = '';
      this.formData.ParentItemId = null;
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  min-width: 70%;
  margin: 0 auto;
}

.left, .right {
  flex: 1;
  padding: 20px;
  justify-content: center;
}

.left {
  background: red;
}

.right {
  background: yellow;
}
</style>
