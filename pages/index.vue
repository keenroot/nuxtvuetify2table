<template>
  <v-app>
    <!-- футер с поиском -->
    <v-card-title>
      <v-text-field
        v-model="searchText"
        append-icon="mdi-magnify"
        label="Поиск"
        single-line
        hide-details
        @keydown.enter="useSearch"
        @keydown.esc="clearSearch"
      />
      <v-btn class="ml-5 mt-3" @click="useSearch">Найти</v-btn>
      <v-btn class="ml-5 mt-3" @click="useSearchLastEdited"
        >Отредаетированный</v-btn
      >
    </v-card-title>
    <!-- основная таблица -->
    <v-data-table
      v-model="selected"
      :headers="headers"
      :items="items"
      item-value="id"
      :search="search"
      show-select
      class="elevation-1"
    >
      <template v-slot:body="{ items, isSelected, select }">
        <tbody>
          <tr v-for="item in items" :key="item.id">
            <td>
              <v-checkbox
                :input-value="isSelected(item)"
                style="margin: 0px; padding: 0px"
                color="#535353"
                hide-details
                @click="select(item, !isSelected(item))"
              />
            </td>
            <td>
              {{ item.title }}
              <v-icon class="ml-2" @click="editItem(item)"> mdi-pencil </v-icon>
            </td>
            <td>
              <v-icon class="mr-2"> mdi-clock-time-four-outline </v-icon>
              {{ item.publishedAt }}
            </td>
            <td>
              <v-icon class="mr-2"> mdi-clock-plus-outline </v-icon>
              {{ item.updatedAt }}
            </td>
            <td>
              <v-icon class="me-2" @click="saveToFile(item)">
                mdi-folder-arrow-up
              </v-icon>
              <v-icon @click="itemDelete(item)"> mdi-delete </v-icon>
            </td>
          </tr>
        </tbody>
      </template>

      <!-- кнопка в футере для удаления выделенных айтемов -->
      <template #footer.prepend>
        <v-icon @click="itemCheckedDelete" class="ml-2"> mdi-delete </v-icon>
      </template>
    </v-data-table>
    <v-dialog v-model="dialog" max-width="700px">
      <v-card class="px-5 pb-5 pt-2" align="right">
        <v-text-field v-model="editedItem.title" />
        <v-textarea v-model="editedItem.content" />
        <v-btn @click="confirmEdit">Сахранить</v-btn>
        <v-btn @click="cancelEdit" class="ml-2">Отмена</v-btn>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      //редактируемый элемент
      editedItemIndex: 0,
      editedItem: { content: '', title: '', updatedAt: '' },
      //оторбражение диалога для редактирования
      dialog: false,
      //поиск для v-data-table
      search: '',
      //промежуточная переменная поиска для реализации кнопки найти и нажатий entr exc
      searchText: '',
      //массви выделенных айтемов
      selected: [],
      //основной массив с айтемами
      items: [],
      //массив для v-data-table с хедерами
      headers: [
        {
          text: 'Название',
          align: 'start',
          sortable: true,
          value: 'title',
        },
        {
          text: 'Опублекованно',
          align: 'start',
          sortable: true,
          value: 'publishedAt',
        },
        {
          text: 'Изменено',
          align: 'start',
          sortable: true,
          value: 'updatedAt',
        },
      ],
    }
  },
  mounted() {
    //получение api
    this.fetchItems()
  },
  methods: {
    //функция для получения api
    async fetchItems() {
      try {
        const response = await axios.get('https://jsonplaceholder.org/posts')
        this.items = response.data
      } catch (e) {
        console.log(e)
      }
    },
    // удаление одного айтема
    itemDelete(item) {
      this.items = this.items.filter((itm) => itm.id !== item.id)
    },
    // удаление выделенных айтемов кнопкой в футере
    itemCheckedDelete() {
      for (let i = 0; i < this.selected.length; i++) {
        this.items = this.items.filter((itm) => itm.id !== this.selected[i].id)
      }
    },
    //поиск по нажатию кнопки найти в хедере или по нажатию entr
    useSearch() {
      this.search = this.searchText
    },
    //поиск последнего отредактированного айтема
    useSearchLastEdited() {
      this.searchText = this.editedItem.title
      this.search = this.editedItem.title
    },
    //отмена поиска по нажатию esc
    clearSearch() {
      this.search = ''
    },
    //получения айтема для редактирования, поиск индекса айтема в общем масстве
    editItem(item) {
      this.dialog = true
      this.editedItem.title = item.title
      this.editedItem.content = item.content
      this.editedItemIndex = this.items.indexOf(item)
    },
    //применение редактирования
    confirmEdit() {
      //получение и форматирование даты
      let currentDate = new Date().toLocaleString().replace(/[.]/g, '/')
      currentDate = currentDate.replace(',', '')
      //присваивание значений вобект айтема в основном массиве по индексу
      this.items[this.editedItemIndex].title = this.editedItem.title
      this.items[this.editedItemIndex].updatedAt = currentDate
      this.items[this.editedItemIndex].content = this.editedItem.content
      this.dialog = false
    },
    //отмена редактирования
    cancelEdit() {
      this.dialog = false
    },
    //сохранение айтема в файл
    saveToFile(item) {
      let data = new Blob([item.title + '\n\n' + item.content], {
        type: 'text/plain',
      })

      let a = document.createElement('a')
      document.body.appendChild(a)
      a.style = 'display: none'

      let url = window.URL.createObjectURL(data)
      a.href = url
      a.download = item.slug
      a.click()
      window.URL.revokeObjectURL(url)
      document.body.removeChild(a)
    },
  },
}
</script>

<style>
/* сдвиг пагинации в лево */
.v-application--is-ltr .v-data-footer__select {
  margin-left: 40px;
  margin-right: 15px;
}
</style>
