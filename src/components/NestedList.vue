<template>
  <button class="mainSort" v-if="nestIndex === 0" @click="toggleSortDirection">
    Zmień kierunek sortowania
  </button>

  <div class="container">
    <Draggable v-model="localItems" item-key="id" class="itemElement" tag="ul">
      <template #item="{ element }">
        <div>
          <div class="liContainer">
            <div class="liLeft">
              <p class = "liTitle">{{ element.title }}</p>
              <form v-if="element.showAddForm" @submit.prevent="submitForm(element)">
                <label :for="'formTitle_' + element.id">Podaj nazwę nowego folderu:
                  <input
                    :id="'formTitle_' + element.id"
                    type="text"
                    v-model="element.formTitle"
                    required
                  />
                </label>
                <button type="submit">Zapisz</button>
              </form>
              <form v-if="element.showEditForm" @submit.prevent="editItem(element)">
                <label for="title">Podaj nowy tytuł:
                  <br><input type="text" id="title" v-model="element.newTitle" required>
                </label><br>
                <button type="submit">Zmień nazwę</button>
              </form>
              <form v-if="element.showEditForm" @submit.prevent="moveItem(element)">
                <label :for="'parentItemId_' + element.id">Przenieś do:
                  <br><select :id="'parentItemId_' + element.id"
                          v-model="element.newParentItemId"
                          required @focus="loadDropdownItems(element.id)">
                    <option value="">Nie przenoś</option>
                    <option v-for="item in dropdownItems" :value="item.id" :key="item.id">
                      {{ item.title }}
                    </option>
                  </select>
                </label><br>
                <button type="submit">Zapisz</button>
              </form>
            </div>
            <div class="liRight">
              <ul class="itemOptions">
                <li>
                  <button @click="toggleAddFormVisibility(element)">Dodaj</button>
                </li>
                <li>
                  <button @click="toggleEditFormVisibility(element)">Edytuj</button>
                </li>
                <li>
                  <button @click="deleteItem(element.id)">Usuń</button>
                </li>
                <li v-if="element.childItems.length > 0">
                  <button @click="toggleChildItemsVisibility(element)">
                    {{ element.showChildItems ? 'Ukryj' : 'Pokaż' }}
                  </button>
                </li>
              </ul>
              <NestedList
                class="NestedElement"
                :items="element.childItems"
                :nestIndex="nestIndex + 1"
                v-model="localSortDirection"
                v-if="element.showChildItems"
              />
            </div>
          </div>
        </div>
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
    nestIndex: {
      type: Number,
      default: 0,
    },
    sortDirection: {
      type: String,
      default: 'asc',
    },
  },

  data() {
    return {
      localItems: [],
      localSortDirection: this.sortDirection,
      dropdownItems: [],
    };
  },
  watch: {
    localSortDirection(newSortDirection) {
      this.$emit('input', newSortDirection);
    },
  },
  mounted() {
    this.localItems = this.items.map((item) => ({ ...item, showChildItems: true }));
  },
  methods: {
    loadDropdownItems(id) {
      axios.get(`https://localhost:44313/api/item/exclude/${id}`)
        .then((response) => {
          this.dropdownItems = response.data.map((item) => ({
            ...item,
            showChildItems: true,
          }));
        })
        .catch((error) => {
          if (error.response && error.response.status === 404) {
            console.log('Nie znaleziono danych.');
          }
        });
    },

    toggleChildItemsVisibility(element) {
      const updatedElement = { ...element };
      updatedElement.showChildItems = !updatedElement.showChildItems;
      this.localItems.splice(this.localItems.indexOf(element), 1, updatedElement);
    },

    toggleSortDirection() {
      this.localSortDirection = this.localSortDirection === 'asc' ? 'desc' : 'asc';
      this.localItems.sort((a, b) => {
        if (this.localSortDirection === 'asc') {
          return a.title.localeCompare(b.title);
        }
        return b.title.localeCompare(a.title);
      });
    },
    toggleAddFormVisibility(element) {
      const updatedElement = { ...element };
      updatedElement.showAddForm = !updatedElement.showAddForm;
      updatedElement.formTitle = '';
      this.localItems.splice(this.localItems.indexOf(element), 1, updatedElement);
    },
    toggleEditFormVisibility(element) {
      const updatedElement = { ...element };
      updatedElement.showEditForm = !updatedElement.showEditForm;
      this.localItems.splice(this.localItems.indexOf(element), 1, updatedElement);
    },

    submitForm(element) {
      const data = {
        ParentItemId: element.id,
        Title: element.formTitle,
      };

      axios
        .post('https://localhost:44313/api/item', data)
        .then(() => {
          this.toggleAddFormVisibility(element);
          window.location.reload();
        })
        .catch((error) => {
          if (error.response && error.response.data && error.response.data.length > 0) {
            const [{ errorMessage }] = error.response.data;
            alert(errorMessage);
          }
          console.error(error);
        });
    },
    editItem(element) {
      const data = {
        Title: element.newTitle,
      };
      console.log(data);
      axios
        .put(`https://localhost:44313/api/item/rename/${element.id}`, data)
        .then((response) => {
          console.log(response.data);
          this.toggleEditFormVisibility(element);
          window.location.reload();
        })
        .catch((error) => {
          console.error(error);
        });
    },

    moveItem(element) {
      if (element.newParentItemId === '') {
        return;
      }

      const data = {
        ParentItemId: element.newParentItemId,
      };
      console.log(data);
      axios
        .put(`https://localhost:44313/api/item/move/${element.id}`, data)
        .then((response) => {
          console.log(response.data);
          this.toggleEditFormVisibility(element);
          window.location.reload();
        })
        .catch((error) => {
          console.error(error);
        });
    },
    deleteItem(itemId) {
      axios
        .delete(`https://localhost:44313/api/item/${itemId}`)
        .then((response) => {
          console.log(response.data);
          window.location.reload();
        })
        .catch((error) => {
          console.error(error);
        });
    },
  },
};
</script>

<style scoped>

button
{
  font-family: 'Roboto', sans-serif;
  padding: 5px;
  margin-bottom: 5px;
}

button:hover
{
  cursor: pointer;
}

.liContainer {
  display: flex;
  flex-direction: row;
  padding: 20px;
  margin: 5px;
  background: #e7e7d0;
  border: 2px #284b55 solid;
  border-radius: 10px;
}

.liLeft,
.liRight {
  flex: 1;
}

.liTitle
{
  font-size: 140%;
  text-decoration: underline;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.itemOptions {
  display: flex;
  flex-direction: row;
  margin: 0 auto;
  justify-content: right;
}

.itemOptions li {
  margin-right: 10px;
  list-style-type: none;
}

</style>
