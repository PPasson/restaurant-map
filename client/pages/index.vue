<template>
  <div>
    <NavBar />
    <div class="container">
      <div class="input-group mb-3 pt-4" style="max-width: 30rem">
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

      <div v-if="loading"><img src="~/assets/foodLoader.gif" /></div>

      <div v-else>
        <div v-for="(result, index) in results" :key="index">
          <b-card class="m-3">
            <div class="fluid">
              <h3>
                {{ result.name }}
              </h3>
              <b-form-rating
                id="rating-inline"
                inline
                value="4"
                no-border
                v-model="result.rating"
                variant="warning"
                readonly
                class="ml-n3"
              ></b-form-rating>
              <span
                v-if="result.opening_hours && result.opening_hours.open_now"
                class="openNow"
                >open now</span
              >
            </div>
            <p>{{ result.formatted_address }}</p>
          </b-card>
        </div>
      </div>
      <div v-if="nextPageToken" class="p-4">
        <button
          class="btn btn-primary"
          type="button"
          @click="loadMoreResults()"
        >
          Load More
        </button>
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
      nextPageToken: null,
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
      this.nextPageToken = response.next_page_token;
      console.log("result = ", this.results);
    },
    async loadMoreResults() {
      this.loading = true;
      const apiKey = process.env.GOOGLE_MAP_API_KEY;
      const response = await this.$axios.$get("/maps/place/textsearch/json", {
        params: {
          pagetoken: this.nextPageToken,
          key: apiKey,
        },
      });
      this.loading = false;
      this.results = this.results.concat(response.results);
      this.nextPageToken = response.next_page_token;
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 1080px;
  display: grid;
  justify-items: center;
}
.openNow {
  color: white;
  text-shadow: 1px 1px 2px black, 0 0 25px green, 0 0 5px darkgreen;
  animation: blinker 0.7s linear infinite;
  margin-right: 1rem;
}
@keyframes blinker {
  50% {
    opacity: 50%;
    text-shadow: 1px 1px 2px black, 0 0 25px yellowgreen, 0 0 5px darkgoldenrod;
  }
}
</style>
