<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
      <NestedList :items="item.children" v-if="item.children" />
    </li>
  </ul>
</template>

<script>
export default {
  name: 'NestedList',
  props: {
    items: Array,
  },
  components: {
    NestedList: {
      // Rekurencyjne wywołanie komponentu
      extends: this,
    },
  },
};
</script>
