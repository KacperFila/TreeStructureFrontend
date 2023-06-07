<template>
  <div class="container">
    <div class="left">
      <form @submit.prevent="submitForm">
        <h2>Utwórz nowy katalog:</h2>
        <label for="title">Nazwa:
          <input type="text" id="title" v-model="formData.Title" required>
        </label>
        <br>
        <button id="createButton" type="submit">Stwórz</button>
      </form>
    </div>

    <div class="right">
      <div v-if="fetchedData.length > 0">
        <NestedList
          :items="fetchedData"
          :nestIndex="0"
          :sortDirection="sortDirection"
        />
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
          this.fetchedData = response.data.map((item) => ({
            ...item,
            showEditForm: false,
            showAddForm: false,

          }));

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
          if (error.response && error.response.data && error.response.data.length > 0) {
            const [{ errorMessage }] = error.response.data;
            alert(errorMessage);
          }
          console.error(error);
        });

      this.formData.Title = '';
      this.formData.ParentItemId = null;
    },
  },
};
</script>

<style scoped>

ul {
  list-style-type: circle;
}

.container {
  display: flex;
  min-width: 70%;
  margin: 0 auto;
  font-family: 'Roboto', sans-serif;
}

.left {
  flex: 0 0 20%;
  padding: 20px;
  justify-content: center;
}

.right {
  flex: 0 0 80%;
  padding: 20px;
  justify-content: center;
}

#createButton {
  margin-top: 10px;
  padding: 5px;
}

button:hover
{
  cursor: pointer;
}

</style>
