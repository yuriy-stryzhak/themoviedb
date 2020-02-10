<template>
  <div id="app">
    <div class="block-preview" style="background-image: url('img/poster-bg.jpg');">
      <div class="mainer">
        <div class="block-play"></div>
        <div class="block-caption">
          <h1>RINGS</h1>EVIL IS REBORN
        </div>
        <div class="block-description">
          Paramount Pictures have just released the first trailer for the upcoming
          Horror movie “Rings“.
        </div>
      </div>
    </div>

    <div class="block-genres">
      <div class="mainer">
        <ul>
          <li
            v-for="genre in genresMenu"
            :key="genre.id"
            :class="{active: genre.id === genreActiveId}"
            @click="getMovies(genre.id); genreActiveId = genre.id"
          >{{genre.name}}</li>
        </ul>
      </div>
    </div>

    <div class="block-grid">
      <div class="mainer">
        <div class="grid-wrapper">
          <div class="grid-cell" v-for="movie in movies" :key="movie.id">
            <div class="block-card">
              <div class="label" v-if="movie.popularity > 60">Popular</div>
              <div
                class="poster"
                style="background-image: url('img/poster-bg.jpg');"
                :style="{'background-image': 'url(https://image.tmdb.org/t/p/w500'+ movie.poster_path +')'}"
              >
                <div class="buttons">
                  <button class="btn">Info</button>
                  <button class="btn">trailer</button>
                </div>
                <button class="favourite" @click="addToFavourite(movie.id)">+</button>
              </div>
              <div class="body">
                <div class="caption">{{movie.title}}</div>
                <div class="genres">{{getGenresString(movie.genre_ids)}}</div>
                <div class="rating">
                  <div class="stars">
                    <span :style="{width: movie.vote_average*10 + '%'}"></span>
                  </div>
                  <div class="amount">{{movie.vote_average}}</div>
                </div>
              </div>
            </div>
          </div>

          <div class="grid-cell empty"></div>
          <div class="grid-cell empty"></div>
        </div>

        <div class="load-more">
          <button class="btn" @click="getMovies(genreActiveId, ++currentPage)">...</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      apiUrl: "https://api.themoviedb.org/",
      apiKey: "5e9941dced73dbfc653f0dfd9357e605",
      genres: [],
      genresMenu: [],
      movies: [],
      genreActiveId: null,
      currentPage: 1
    };
  },
  methods: {
    getGenres() {
      fetch(
        this.apiUrl +
          "3/genre/movie/list?api_key=" +
          this.apiKey +
          "&language=ru"
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          this.genres = result.genres;
          this.genresMenu = result.genres.slice(0, 6);
          this.genreActiveId = result.genres[0].id;
          this.getMovies(this.genreActiveId, this.currentPage);
        });
    },
    getMovies(genreId, page) {
      fetch(
        this.apiUrl +
          "3/discover/movie?api_key=" +
          this.apiKey +
          "&language=ru&sort_by=popularity.desc&include_adult=false&include_video=false&page=" +
          page +
          "&with_genres=" +
          genreId
      )
        .then(result => {
          return result.json();
        })
        .then(result => {
          if (page > 1) {
            this.movies = this.movies.concat(result.results);
          } else {
            this.movies = result.results;
          }
        });
    },
    getGenresString(ids) {
      let result = ``;
      for (let id of ids) {
        for (let genre of this.genres) {
          if (id === genre.id) {
            result += ", " + genre.name;
          }
        }
      }
      return result.slice(2, result.length);
    },
    addToFavourite(id) {
      const favouriteList = localStorage.getItem("favourite");
      const currentList = JSON.parse(favouriteList) || [];
      if (favouriteList) {
        if (currentList.indexOf(id) < 0) {
          currentList.push(id);
        } else {
          currentList.splice(currentList.indexOf(id), 1);
        }
      } else {
        currentList.push(id);
      }
      localStorage.setItem("favourite", JSON.stringify(currentList));
    }
  },
  created: function() {
    this.getGenres();
  }
};
</script>

<style lang="scss">
@import "./src/styles/styles.scss";
</style>
