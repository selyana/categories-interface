<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="productsFiltered"
      class="elevation-3 pa-5"
      :single-expand="singleExpand"
      item-key="title"
      show-expand
      hide-default-footer
    >
      <template #expanded-item="{ headers, item }">
        <td :colspan="headers.length" class="pa-5">
          {{ item.description }}
        </td>
      </template>
      <template #[`item.actions`]="{ item }">
        <v-row>
          <v-col>
            <v-autocomplete
              v-model="item.category"
              :items="categories"
              item-text="title"
              outlined
              dense
              label="Выберите категорию"
              class="mt-5"
            />
          </v-col>
          <v-col>
            <v-btn
              elevation="2"
              class="mt-5"
              @click="setCategory(item)"
            >
              Назначить
            </v-btn>
          </v-col>
        </v-row>
      </template>
    </v-data-table>
    <v-pagination
      v-model="page"
      class="mt-3"
      :length="Math.ceil(totalCount / pageSize)"
      :total-visible="7"
      @input="next"
    />
  </div>
</template>

<script>
import env from '~/env.js'

export default {
  async asyncData ({ query, $http }) {
    if (query.page) {
      const products = await $http.$get(`api/admin/products?page=${query.page}`)
      const productsFiltered = products['hydra:member']
      return { productsFiltered }
    } else {
      const products = await $http.$get('api/admin/products?page=1')
      const productsFiltered = products['hydra:member']
      return { productsFiltered }
    }
  },
  data () {
    return {
      categories: [],
      singleExpand: true,
      totalCount: null,
      page: 1,
      pageSize: 10,
      headers: [
        { text: 'Title', value: 'title' },
        { text: 'Category', value: 'actions' }
      ]
    }
  },
  async fetch () {
    const requestProducts = 'admin/products?page=1'
    const productsInfo = await fetch(env.fetchUri + requestProducts).then(res => res.json())
    this.totalCount = productsInfo['hydra:totalItems']
    const requestCategories = 'admin/categories'
    const categoriesInfo = await fetch(env.fetchUri + requestCategories).then(res => res.json())
    this.categories = categoriesInfo['hydra:member']
  },
  watch: {
    $route () {
      this.$nuxt.refresh()
    }
  },
  methods: {
    next (page) {
      this.$router.push({ path: this.$route.path, query: { page } })
    },
    setCategory (item) {
      console.log(item.category)
      console.log(item)
    }
  }
}
</script>
