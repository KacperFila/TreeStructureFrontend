<!--suppress VueUnrecognizedSlot -->
<template>
  <div class="container">
  <Draggable
    v-model="localItems"
    item-key="id"
    class="myList"
    tag="ul">
    <template #item="{element}">
      <li>
        {{ element.title }}
        <NestedList class="NestedElement" :items="element.childItems"
                    v-if="element.childItems && element.childItems.length > 0" />
      </li>
    </template>
  </Draggable>
  </div>
</template>

<script>
import Draggable from 'vuedraggable';

export default {
  name: 'NestedList',
  components: {
    Draggable,
  },
  props: {
    items: {
      type: Array,
    },
  },
  data() {
    return {
      localItems: [],
      drag: false,
      draggableOptions: {
        group: 'nested-list',
      },
    };
  },
  mounted() {
    this.localItems = [...this.items]; // Tworzenie kopii tablicy items
  },
};
</script>

<style scoped>
ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
  width: fit-content;
}

li {
  width: 100%;
  background-color: #f2f2f2;
  padding:10px;
  border: black solid 2px;
}

.NestedElement > li{
  background: red;
  margin-left: 30px;
}
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
