<template>
  <div class="container">
    <!-- 네비게이션 바 -->
    <nav class="tmdb-navbar">
      <span class="tmdb-logo">TMDB <span class="logo-dot"></span></span>
      <a href="#">영화</a>
      <a href="#">TV 프로그램</a>
      <a href="#">인물</a>
      <a href="#">More</a>
    </nav>

    <!-- 정렬/검색 바 -->
    <div class="top-bar">
      <div class="sort-bar">
        <button @click="sortBy('rating')">평점순</button>
        <button @click="sortBy('title')">가나다순</button>
        <button @click="sortBy('date')">최신순</button>
      </div>
      <input
        v-model="searchText"
        @input="filterMovies"
        type="text"
        class="search-input"
        placeholder="영화 제목 검색"
        autocomplete="off"
      />
    </div>

    <!-- 영화 카드 목록 -->
    <div class="movies-grid">
      <div
        class="movie-card"
        v-for="movie in movies"
        :key="movie.id"
        @click="openDetail(movie)"
      >
        <div class="poster-area">
          <img
            class="poster"
            :src="getPosterUrl(movie.poster_path)"
            :alt="movie.title"
          />
        </div>
        <div class="card-info">
          <span class="movie-title" :title="movie.title">{{ movie.title }}</span>
          <span class="movie-rating">{{ movie.vote_average }}</span>
        </div>
      </div>
    </div>

    <!-- 영화 상세 모달 -->
    <div v-if="detailMovie" class="movie-detail-modal" @click.self="closeDetail">
      <div class="detail-content">
        <button class="close-btn" @click="closeDetail">✕</button>
        <img
          class="detail-poster"
          :src="getPosterUrl(detailMovie.poster_path)"
          :alt="detailMovie.title"
        />
        <div class="detail-info">
          <h2>{{ detailMovie.title }}</h2>
          <p class="detail-rating">평점: {{ detailMovie.vote_average }}</p>
          <p class="detail-date">개봉일: {{ detailMovie.release_date }}</p>
          <div class="detail-overview">{{ detailMovie.overview }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      movies: [],
      rawMovies: [],
      detailMovie: null,
      searchText: "",
    };
  },
  mounted() {
    this.fetchMovies();
  },
  methods: {
    async fetchMovies() {
      const apiKey = "bf70d26cea38b2e84c7c7f4122b5eb9c"; // 실제 API 키로 교체!
      const url = `https://api.themoviedb.org/3/movie/popular?api_key=${apiKey}&language=ko-KR&page=1`;
      const response = await axios.get(url);
      this.movies = response.data.results;
      this.rawMovies = [...response.data.results];
    },
    getPosterUrl(path) {
      return path
        ? `https://image.tmdb.org/t/p/w500${path}`
        : "https://via.placeholder.com/350x520?text=No+Image";
    },
    sortBy(type) {
      if (type === "rating") {
        this.movies.sort((a, b) => b.vote_average - a.vote_average);
      } else if (type === "title") {
        this.movies.sort((a, b) =>
          a.title.localeCompare(b.title, "ko", { sensitivity: "base" })
        );
      } else if (type === "date") {
        this.movies.sort((a, b) =>
          (b.release_date || "").localeCompare(a.release_date || "")
        );
      }
    },
    openDetail(movie) {
      this.detailMovie = movie;
    },
    closeDetail() {
      this.detailMovie = null;
    },
    filterMovies() {
      if (this.searchText.trim() === "") {
        this.movies = [...this.rawMovies];
      } else {
        const keyword = this.searchText.trim().toLowerCase();
        this.movies = this.rawMovies.filter((m) =>
          m.title.toLowerCase().includes(keyword)
        );
      }
    },
  },
};
</script>

<style scoped>
.container {
  background: #f4f7fc;
  min-height: 100vh;
}
.tmdb-navbar {
  display: flex;
  align-items: center;
  padding: 0 28px;
  height: 64px;
  background: #1b375c;
  color: #fff;
  font-weight: 500;
  gap: 34px;
}
.tmdb-logo {
  font-size: 1.3em;
  font-weight: 700;
  letter-spacing: 0.5px;
  margin-right: 19px;
  display: flex;
  align-items: center;
}
.logo-dot {
  display: inline-block;
  width: 16px;
  height: 8px;
  background: #39c5a5;
  border-radius: 6px;
  margin-left: 7px;
}
.tmdb-navbar a {
  color: #fff;
  text-decoration: none;
  margin-right: 16px;
  transition: color 0.2s;
}
.tmdb-navbar a:hover {
  color: #39c5a5;
}
/* 정렬/검색바 */
.top-bar {
  max-width: 1200px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 20px auto 2px;
  gap: 14px;
}
.sort-bar {
  display: flex;
  gap: 9px;
}
.sort-bar button {
  background: #39c5a5;
  color: #fff;
  border: none;
  border-radius: 6px;
  padding: 8px 15px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.17s;
}
.sort-bar button:hover {
  background: #1b375c;
}
.search-input {
  background: #fff;
  border: 1.5px solid #39c5a5;
  border-radius: 7px;
  padding: 8px 17px;
  min-width: 180px;
  font-size: 1em;
  color: #424874;
  box-sizing: border-box;
  transition: border 0.13s;
}
.search-input:focus {
  border-color: #1b375c;
  outline: none;
}
.movies-grid {
  margin: 36px auto 0;
  max-width: 1200px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 46px 32px;
}
.movie-card {
  background: #fff;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 28px rgba(27,55,92,0.10);
  display: flex;
  flex-direction: column;
  min-height: 460px;
  max-width: 290px;
  transition: box-shadow 0.18s, transform 0.18s;
  cursor: pointer;
}
.movie-card:hover {
  box-shadow: 0 14px 40px rgba(27,55,92,0.26);
  transform: translateY(-12px) scale(1.05);
  z-index: 2;
}
.poster-area {
  width: 100%;
  height: 390px;
  background: #ededed;
  display: flex;
  align-items: center;
  justify-content: center;
}
.poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.card-info {
  width: 100%;
  background: #424874; 
  height: 70px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  box-sizing: border-box;
  padding: 0 22px;
  overflow: hidden;
}
.movie-title {
  font-size: 1.12em;
  font-weight: 500;
  max-width: 160px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #fff;
}
.movie-rating {
  font-size: 1em;
  font-weight: 600;
  background-color: #39c5a5;
  color: #2d4569;
  padding: 7px 19px;
  border-radius: 12px;
  margin-left: 18px;
  flex-shrink: 0;
}
/* 상세 모달 */
.movie-detail-modal {
  position: fixed;
  top: 0; left: 0;
  width: 100vw; height: 100vh;
  background: rgba(36,45,95,0.36);
  display: flex; align-items: center; justify-content: center;
  z-index: 50;
}
.detail-content {
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 8px 40px rgba(27,55,92,0.20);
  padding: 38px 32px 30px;
  min-width: 340px; max-width: 540px;
  display: flex; flex-direction: column; align-items: center;
  position: relative;
}
.close-btn {
  position: absolute; top: 19px; right: 21px;
  background: none; border: none; font-size: 1.7em; color: #424874; cursor: pointer;
  transition: color 0.2s;
}
.close-btn:hover {
  color: #39c5a5;
}
.detail-poster {
  width: 208px; height: 310px;
  object-fit: cover; border-radius: 9px;
  box-shadow: 0 2px 12px #aab3c5;
  margin-bottom: 16px;
}
.detail-info {
  text-align: left;
  width: 100%;
}
.detail-info h2 {
  font-size: 1.23em;
  margin-bottom: 9px;
  font-weight: bold;
}
.detail-rating, .detail-date {
  font-size: 1em;
  margin-bottom: 5px;
  color: #39c5a5;
}
.detail-overview {
  margin-top: 13px;
  font-size: 1.03em;
  color: #424874;
  line-height: 1.52;
}
</style>
