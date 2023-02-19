<template>
  <div>
    <Cover />
    <div class="container">
      <div
        class="input-group mb-3 mt-n5 p-4 pill-card"
        style="max-width: 30rem"
      >
        <input
          type="text"
          class="form-control"
          placeholder="Search for restaurants"
          v-model="keyword"
          @keydown.enter="searchRestaurants()"
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
            <a
              :href="
                'https://www.google.com/maps/place/?q=place_id:' +
                result.place_id
              "
              target="_blank"
              >Let's go ↗</a
            >
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
    //search function: work when user press ENTER key or click search button
    async searchRestaurants() {
      //turn loader on while getting data
      this.loading = true;

      //define API key by read from .env file
      const apiKey = process.env.GOOGLE_MAP_API_KEY;

      //get data from google map api
      const response = await this.$axios.$get("/maps/place/textsearch/json", {
        params: {
          query: this.keyword,
          type: "restaurant",
          language: "en",
          key: apiKey,
        },
        mode: "cors",
      });

      //turn loader off and set results from response to results
      this.loading = false;
      this.results = response.results;

      //set nextPageToken in case user want to get more result
      this.nextPageToken = response.next_page_token;
    },

    //load more function: in case user want to get more result
    async loadMoreResults() {
      this.loading = true;

      //define API key by read from .env file
      const apiKey = process.env.GOOGLE_MAP_API_KEY;

      //get more data by use page token that we already set on previous function
      const response = await this.$axios.$get("/maps/place/textsearch/json", {
        params: {
          pagetoken: this.nextPageToken,
          key: apiKey,
        },
      });
      //turn loader off
      this.loading = false;

      //concat more results with previous results
      this.results = this.results.concat(response.results);

      //set nextPageToken in case user want to get more result
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
.pill-card {
  background-color: #ffff;
  border-radius: 16px;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
}
</style>
