<template>
  <div>
    <div v-if="fetchedData.length > 0">
      <NestedList :items="fetchedData" />
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
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      axios
        .get('https://localhost:44313/api/item')
        .then((response) => {
          this.fetchedData = response.data;
          console.log(this.fetchedData);
        })
        .catch((error) => {
          console.error(error);
        });
    },
  },
};
</script>
