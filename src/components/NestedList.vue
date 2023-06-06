<template>
  <button class="mainSort" v-if="buttonIndex === 0 && !drag" @click="toggleSortDirection">
    ASC/DESC
  </button>

  <div class="container">

      <Draggable
        v-model="localItems"
        item-key="id"
        class="myList"
        tag="ul"
        :options="draggableOptions"
      >
        <template #item="{ element }">
          <li>
            <div class="liTop">
              {{ element.title }}
              <form @submit.prevent="editItem(element.id)">
                <label for="parentItemId">Parent Item Id:
                  <input type="text" id="parentItemId" v-model="parentItemId" required>
                </label>

                <label for="title">Title:
                  <input type="text" id="title" v-model="title" required>
                </label>
                <button type="submit">Wyślij</button>
              </form>

            </div>

            <div class ="liBottom">
              <button @click="deleteItem(element.id)">Delete catalog</button>

              <form @submit.prevent="submitForm(element.id)">
                <label :for="'formTitle_' + element.id">New element:
                  <input
                    :id="'formTitle_' + element.id"
                    type="text"
                    v-model="formTitle"
                    required
                  />
                </label>
                <button type="submit">Add</button>
              </form>

              <NestedList
                class="NestedElement"
                :items="element.childItems"
                :button-index="buttonIndex + 1"
                v-model="localSortDirection"
              />
            </div>
          </li>
        </template>
      </Draggable>
    </div>
</template>

<script>
import Draggable from 'vuedraggable';
import axios from 'axios';

export default {
  name: 'NestedList',
  components: {
    Draggable,
  },
  props: {
    items: {
      type: Array,
    },
    buttonIndex: {
      type: Number,
      default: 0,
    },
    value: {
      type: String,
      default: 'asc',
    },
  },
  data() {
    return {
      localItems: [],
      drag: false,
      draggableOptions: {
        group: 'nested-list',
        ghostClass: 'drag-ghost',
        chosenClass: 'drag-chosen',
        removeOnSpill: true,
      },
      localSortDirection: this.value,
    };
  },
  watch: {
    localSortDirection(newSortDirection) {
      this.$emit('input', newSortDirection);
    },
  },
  mounted() {
    this.localItems = [...this.items];
  },
  methods: {
    toggleSortDirection() {
      this.localSortDirection = this.localSortDirection === 'asc' ? 'desc' : 'asc';
      this.localItems.sort((a, b) => {
        if (this.localSortDirection === 'asc') {
          return a.title.localeCompare(b.title);
        }
        return b.title.localeCompare(a.title);
      });
    },
    submitForm(parentItemId) {
      const data = {
        ParentItemId: parentItemId,
        Title: this.formTitle,
      };
      axios
        .post('https://localhost:44313/api/item', data)
        .then((response) => {
          console.log(response.data);
          // Obsługa odpowiedzi
        })
        .catch((error) => {
          console.error(error);
          // Obsługa błędu
        });
      window.location.reload();
      this.formTitle = '';
    },
    editItem(Id) {
      const data = {
        ParentItemId: this.parentItemId,
        Title: this.title,
      };
      console.log('Wartość zmiennej Id:', Id);
      axios
        .put(`https://localhost:44313/api/item/${Id}`, data)
        .then((response) => {
          console.log(response.data);
          // Obsługa odpowiedzi
          window.location.reload();
        })
        .catch((error) => {
          console.error(error);
          // Obsługa błędu
        });
    },
    deleteItem(itemId) {
      axios
        .delete(`https://localhost:44313/api/item/${itemId}`)
        .then((response) => {
          console.log(response.data);
          // Obsługa odpowiedzi
        })
        .catch((error) => {
          console.error(error);
          // Obsługa błędu
        });
      window.location.reload();
      this.formTitle = '';
    },
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
  display: flex;
  flex-direction: column;
}

.liTop, .liBottom {
  width: 100%;
}

.liTop {
  background-color: #f2f2f2;
  padding: 10px;
  border: black solid 1px;
}

.liBottom {
  background-color: lightblue;
  padding: 10px;
  border: black solid 1px;
  border-top:0;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

</style>
