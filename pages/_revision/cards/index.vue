<template>
  <div>
    <b-container v-if="cards">
      <h1 class="mt-4 text-primary">
        カード一覧 ({{ $route.params.revision }})
      </h1>
      <b-list-group class="mt-4">
        <b-list-group-item
          v-for="card in cards"
          :key="card.id"
          :to="`/AG${card.revision_id}/card/${card.literal_id}`"
          class="d-flex justify-content-between align-items-center"
        >
          {{ card.name_ja }}
          <CardIdBadge :card="card" />
        </b-list-group-item>
      </b-list-group>
      <div class="mt-3 mb-5">
        <b-pagination
          v-model="currentPage"
          :total-rows="meta.total"
          :per-page="meta.per_page"
          align="center"
          @input="solvePaginationLink"
        ></b-pagination>
      </div>
      <div class="searchButtonBox">
        <b-button
          :to="`/${$route.params.revision}/cards/search`"
          variant="info"
          size="lg"
          ><b-icon-search
        /></b-button>
      </div>
    </b-container>
  </div>
</template>
<script>
export default {
  async asyncData({ payload, params, error, $axios }) {
    if (payload) {
      return {
        cards: payload.cards,
        meta: payload.meta,
      }
    }

    if (params.revision !== 'AG1' && params.revision !== 'AG2') {
      error({ statusCode: 404, message: 'Page not found' })
      return false
    }

    const page = params.page || '1'
    const data = await $axios
      .$get(`/${params.revision}/cards`, {
        params: {
          page,
        },
      })
      .catch((_) => {
        error({ statusCode: 404, message: 'Page not found' })
        return false
      })
    return {
      cards: data.cards,
      meta: data.meta,
    }
  },

  head() {
    const title = `カード一覧 (${this.$route.params.revision})`
    return {
      title,
      meta: [
        {
          hid: 'og:title',
          property: 'og:title',
          content: title,
        },
      ],
    }
  },

  computed: {
    currentPage: {
      get() {
        return this.meta.current_page || 1
      },
      set(_) {},
    },
  },

  methods: {
    solvePaginationLink(page) {
      this.$router.push(`/${this.$route.params.revision}/cards/${page}`)
    },
  },
}
</script>
<style scoped>
.searchButtonBox {
  position: fixed;
  right: 16px;
  bottom: 16px;
  z-index: 10;
}
</style>
