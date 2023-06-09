<script>
import { state } from "../state.js";
import axios from "axios";
import LangFlag from "vue-lang-code-flags";
import lang_db from "../data/lang.json";

export default {
  name: "ProductItem",
  components: { LangFlag },

  data() {
    return {
      state,
      cast: [],
    };
  },

  props: {
    title: String,
    original: String,
    lang: String,
    vote: Number,
    poster: String,
    overview: String,
    type: String,
    id: Number,
    genre_ids: Array,
  },

  methods: {
    // If lang is inclued in lang.notSupported -> supported() return false
    supported(lang) {
      return !lang_db.notSupported.includes(lang);
    },

    //Builds the full path where the poster is located
    srcPath(endPath) {
      return `https://image.tmdb.org/t/p/w342/${endPath}`;
    },

    // Return the number of star, from 1 to 5, corresponding to the vote
    fillStars(vote) {
      vote = vote ? vote : 0; // put to zero if vote is null
      return Math.round(vote / 2);
    },

    //Return the number of void star, complementair to 5
    emptyStars(vote) {
      return 5 - this.fillStars(vote);
    },

    getCast() {
      const url = state.API_URL_BASE + this.type + "/" + this.id + "/credits?" + state.API_URL_KEY;
      //console.log(url);
      axios
        .get(url)
        .then((response) => {
          this.cast = response.data.cast;
        })
        .catch((err) => {
          console.log(err);
          console.error(err.message);
        });
    },

    // Return the name from genre's id
    toGenre(genre_id) {
      let genres = this.getGenres(this.type);
      const genre = genres.find((genre) => genre.id === genre_id).name;
      return genre;
    },

    // Return the list of genres by product type
    getGenres(type) {
      if (type == "movie") {
        return state.moviesGenres;
      } else {
        return state.seriesGenres;
      }
    },
  },

  created() {
    this.getCast();
  },
};
</script>
<template>
  <div class="card position-relative">
    <img v-if="poster" :src="srcPath(poster)" class="card-img" :alt="title" />
    <div class="badge position-absolute top-0 start-0 mt-1 ms-1" :class="type == 'tv' ? 'bg-danger' : 'bg-primary'">{{ type }}</div>
    <div class="card-body h-100 w-100 text-center">
      <h5 class="card-title mb-0">{{ title }}</h5>
      <!-- /.ms_title -->
      <p class="card-text mb-2">{{ original }}</p>
      <!-- /.ms_original -->
      <div class="ms_lang mb-2">
        <span v-if="supported(lang)">
          <LangFlag :iso="lang" />
        </span>
        <span v-else>{{ lang }} - not supported</span>
      </div>
      <!-- /.ms_lang -->
      <div class="ms_vote mb-2">
        <span v-for="n in fillStars(vote)" class="fa-solid fa-star"></span>
        <span v-for="n in emptyStars(vote)" class="fa-regular fa-star"></span>
      </div>
      <!-- /.ms_vote -->
      <p class="card-text ms_overview">{{ overview }}</p>
      <!-- /.ms_overview -->
      <div class="ms_actor mb-2">
        <h6 class="ms_title fw-bold mb-0">Actors: </h6>
        <span class="ms_list" v-for="actor in cast.slice(0, 5)">{{ actor.original_name }}</span>
      </div>
      <!-- /.ms_actor -->
      <div class="ms_genres mb-2">
        <h6 class="ms_title fw-bold mb-0">Genres: </h6>
        <span class="ms_list" v-for="genre_id in genre_ids">{{ toGenre(genre_id) }}</span>
      </div>
      <!-- /.ms_genres -->
    </div>
    <!-- /.body -->
  </div>
  <!-- /.card -->
</template>

<style lang="scss" scoped>
@use "../assets/scss/partials/variables" as *;

.card {
  position: relative;
  aspect-ratio: 0.66;
  background-color: $dark;

  .badge {
    z-index: 2;
  }

  .card-img {
    position: relative;
    z-index: 2;
    height: 100%;
    object-fit: cover;
  }

  .card-body {
    position: absolute;
    z-index: 1;
    overflow-y: auto;
    color: $light;

    /* width */
    &::-webkit-scrollbar {
      width: 20px;
    }

    /* Handle */
    &::-webkit-scrollbar-thumb {
      border-radius: 40px;
      box-shadow: inset 0 0 20px 20px $darkless;
      border: solid 6px transparent;
    }

    .ms_actor {
      font-size: 0.9rem;
    }

    .ms_genres {
      font-size: 0.9rem;
    }

    .ms_overview {
      font-style: italic;
      font-size: 0.8rem;
      text-align: justify
    }

    .ms_list:after {
      content: ", ";
    }

    .ms_list:last-child:after {
      content: none;
    }
  }
}

.card:hover .card-img {
  filter: brightness(0.4);
}

.card:hover .card-body {
  z-index: 2;
}

.card:hover .badge {
  z-index: 1;
}
</style>
