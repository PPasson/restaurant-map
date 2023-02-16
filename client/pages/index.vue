<template>
  <div>
    <NavBar />
    <div class="container">
      <div class="input-group mb-3 pt-2">
        <input
          type="text"
          class="form-control"
          placeholder="Search for restaurants"
          v-model="keyword"
        />
        <div class="input-group-append">
          <button
            class="btn btn-primary"
            type="button"
            @click="searchRestaurants()"
          >
            Search
          </button>
        </div>
      </div>

      <div v-if="loading">Loading...</div>

      <div v-else>
        <div v-for="(result, index) in results" :key="index">
          <b-card class="m-3">
            <h3>
              {{ result.name }}
              <b-form-rating
                id="rating-inline"
                inline
                value="4"
                no-border
                v-model="result.rating"
                variant="warning"
                readonly
              ></b-form-rating>
            </h3>
            <p>{{ result.formatted_address }}</p>
          </b-card>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      keyword: "",
      results: [],
      loading: false,
    };
  },
  methods: {
    async searchRestaurants() {
      this.loading = true;
      const apiKey = process.env.GOOGLE_MAP_API_KEY;
      const response = await this.$axios.$get("/maps/place/textsearch/json", {
        params: {
          query: this.keyword,
          type: "restaurant",
          language: "en",
          key: apiKey,
        },
      });
      this.loading = false;
      this.results = response.results;
      console.log("result = ", this.results);
    },
  },
};
</script>

<style scoped>
  .container{
    max-width: 1080px;
  }
</style>
