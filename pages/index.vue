<template>
  <v-app>
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
    </v-card-title>
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
              <v-icon class="ml-2"> mdi-pencil </v-icon>
            </td>
            <td>
              <v-icon class="mr-2"> mdi-clock-time-four-outline </v-icon>
              {{ item.publishedAt }}
            </td>
            <td>
              <v-icon class="mr-2"> mdi-clock-plus-outline </v-icon>
              <mdicon name="hamburger" />
              {{ item.updatedAt }}
            </td>
            <td>
              <v-icon class="me-2"> mdi-folder-arrow-up </v-icon>
              <v-icon @click="itemDelete(item)"> mdi-delete </v-icon>
            </td>
          </tr>
        </tbody>
      </template>
      <template #footer.prepend>
        <v-icon @click="itemCheckedDelete" class="ml-2"> mdi-delete </v-icon>
      </template>
    </v-data-table>
  </v-app>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      search: '',
      searchText: '',
      selected: [],
      items: [],
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
    this.fetchItems()
  },
  methods: {
    async fetchItems() {
      try {
        const response = await axios.get('https://jsonplaceholder.org/posts')
        this.items = response.data
      } catch (e) {
        console.log(e)
      }
    },
    itemDelete(item) {
      this.items = this.items.filter((itm) => itm.id !== item.id)
    },
    itemCheckedDelete() {
      for (let i = 0; i < this.selected.length; i++) {
        this.items = this.items.filter((itm) => itm.id !== this.selected[i].id)
      }
    },
    useSearch() {
      this.search = this.searchText
    },
    clearSearch() {
      this.search = ''
    },
  },
}
</script>

<style>
.v-application--is-ltr .v-data-footer__select {
  margin-left: 40px;
  margin-right: 15px;
}
</style>
