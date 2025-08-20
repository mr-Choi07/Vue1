<template>
  <div class="container">
    <nav class="tmdb-navbar">
      <span class="tmdb-logo">TMDB <span class="logo-dot"></span></span>
      <a href="#" :class="{ active: tab==='movie' }" @click.prevent="changeTab('movie')">영화</a>
      <a href="#" :class="{ active: tab==='tv' }" @click.prevent="changeTab('tv')">TV 프로그램</a>
      <a href="#">인물</a>
      <a href="#">More</a>
    </nav>

    <div class="main-section">
      <div class="top-bar">
        <div class="sort-bar">
          <button @click="sortBy('rating')">평점순</button>
          <button @click="sortBy('title')">가나다순</button>
          <button @click="sortBy('date')">최신순</button>
        </div>

        <input
          v-model="searchText"
          @input="handleSearch"
          type="text"
          class="search-input"
          :placeholder="tab === 'movie' ? '영화 제목 검색' : 'TV 제목 검색'"
          autocomplete="off"
        />
      </div>

      <div class="movies-grid">
        <div
          class="movie-card"
          v-for="item in items"
          :key="item.id"
          @click="openDetail(item)"
        >
          <div class="poster-wrap">
            <img
              class="poster"
              :src="getPosterUrl(item.poster_path)"
              :alt="getTitle(item)"
            />
          </div>
          <div class="card-info">
            <span class="movie-title" :title="getTitle(item)">{{ getTitle(item) }}</span>
            <span class="movie-rating">{{ item.vote_average }}</span>
          </div>
        </div>
      </div>

      <div v-if="!searchText && totalPages > 1" class="pagination-bar">
        <button :disabled="page === 1" class="page-btn" @click="changePage(page - 1)">이전</button>
        <button
          v-for="num in pageNums"
          :key="num"
          class="page-btn"
          :class="{ active: num === page }"
          @click="changePage(num)"
        >
          {{ num }}
        </button>
        <button :disabled="page === totalPages" class="page-btn" @click="changePage(page + 1)">다음</button>
      </div>

      <div v-if="searchText && searchPageCount > 1" class="pagination-bar">
        <button :disabled="searchPage === 1" class="page-btn" @click="changeSearchPage(searchPage - 1)">이전</button>
        <button
          v-for="num in searchPageNums"
          :key="num"
          class="page-btn"
          :class="{ active: num === searchPage }"
          @click="changeSearchPage(num)"
        >
          {{ num }}
        </button>
        <button :disabled="searchPage === searchPageCount" class="page-btn" @click="changeSearchPage(searchPage + 1)">다음</button>
      </div>
    </div>

    <div v-if="detailItem" class="movie-modal" @click.self="closeModal">
      <div class="modal-content">
        <button class="close-btn" @click="closeModal">✕</button>

        <div class="modal-left">
          <img class="poster-large" :src="getPosterUrl(detailItem.poster_path)" :alt="getTitle(detailItem)" />
        </div>

        <div class="modal-right">
          <div class="movie-info">
            <h2>{{ getTitle(detailItem) }}</h2>
            <p class="score">평점: {{ detailItem.vote_average }}</p>
            <p class="date">
              {{ tab === 'movie' ? '개봉일: ' + detailItem.release_date : '방영일: ' + detailItem.first_air_date }}
            </p>
            <p class="overview">{{ detailItem.overview }}</p>
          </div>

          <div class="trailer-section">
            <h3>예고편</h3>
            <div v-if="trailerKey" class="trailer-video">
              <iframe
                :src="`https://www.youtube.com/embed/${trailerKey}`"
                frameborder="0"
                allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                allowfullscreen
                title="트레일러 영상"
              ></iframe>
            </div>
            <div v-else class="no-trailer">예고편이 없습니다.</div>
          </div>
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
      tab: "movie",
      items: [],
      detailItem: null,
      searchText: "",
      page: 1,
      totalPages: 1,
      searchPage: 1,
      searchPageCount: 1,
      trailerKey: null,
      apiKey: "bf70d26cea38b2e84c7c7f4122b5eb9c",
    };
  },
  computed: {
    pageNums() {
      const maxBtn = 5;
      let start = Math.max(this.page - Math.floor(maxBtn / 2), 1);
      let end = start + maxBtn - 1;
      if (end > this.totalPages) {
        end = this.totalPages;
        start = Math.max(end - maxBtn + 1, 1);
      }
      const nums = [];
      for (let i = start; i <= end; i++) nums.push(i);
      return nums;
    },
    searchPageNums() {
      const maxBtn = 5;
      let start = Math.max(this.searchPage - Math.floor(maxBtn / 2), 1);
      let end = start + maxBtn - 1;
      if (end > this.searchPageCount) {
        end = this.searchPageCount;
        start = Math.max(end - maxBtn + 1, 1);
      }
      const nums = [];
      for (let i = start; i <= end; i++) nums.push(i);
      return nums;
    },
  },
  watch: {
    tab() {
      this.page = 1;
      this.searchText = "";
      this.fetchItems();
    },
    page() {
      if (!this.searchText) this.fetchItems();
    },
  },
  mounted() {
    this.fetchItems();
  },
  methods: {
    async fetchItems() {
      const type = this.tab === "movie" ? "movie" : "tv";
      const url = `https://api.themoviedb.org/3/${type}/popular?api_key=${this.apiKey}&language=ko-KR&page=${this.page}`;
      try {
        const res = await axios.get(url);
        this.items = res.data.results;
        this.totalPages = res.data.total_pages;
        this.detailItem = null;
        this.trailerKey = null;
      } catch (error) {
        console.error(error);
      }
    },
    async runSearch() {
      if (!this.searchText.trim()) {
        this.fetchItems();
        return;
      }
      const type = this.tab === "movie" ? "movie" : "tv";
      const url = `https://api.themoviedb.org/3/search/${type}`;
      try {
        const res = await axios.get(url, {
          params: {
            api_key: this.apiKey,
            query: this.searchText,
            language: "ko-KR",
            page: this.searchPage,
          },
        });
        this.items = res.data.results;
        this.searchPageCount = res.data.total_pages;
        this.detailItem = null;
        this.trailerKey = null;
      } catch (error) {
        console.error(error);
      }
    },
    sortBy(type) {
      if (this.items.length <= 1) return;
      const getTitle = (item) => (this.tab === "movie" ? item.title : item.name);
      switch (type) {
        case "rating": {
          const sorted = this.items.sort((a, b) => b.vote_average - a.vote_average);
          this.items = [...sorted];
          break;
        }
        case "title": {
          const sorted = this.items.sort((a, b) =>
            getTitle(a).localeCompare(getTitle(b), "ko", { sensitivity: "base" })
          );
          this.items = [...sorted];
          break;
        }
        case "date": {
          const getDate = (item) => (this.tab === "movie" ? item.release_date : item.first_air_date);
          const sorted = this.items.sort((a, b) =>
            (getDate(b) || "").localeCompare(getDate(a) || "")
          );
          this.items = [...sorted];
          break;
        }
      }
    },
    async handleSearch() {
      this.searchPage = 1;
      this.runSearch();
    },
    async openDetail(item) {
      this.detailItem = item;
      this.trailerKey = null;
      const type = this.tab === "movie" ? "movie" : "tv";
      const url = `https://api.themoviedb.org/3/${type}/${item.id}/videos?api_key=${this.apiKey}&language=ko-KR`;
      try {
        const res = await axios.get(url);
        const trailers = res.data.results.filter(
          (v) => v.site === "YouTube" && (v.type === "Trailer" || v.type === "Teaser")
        );
        this.trailerKey = trailers.length ? trailers[0].key : null;
      } catch (error) {
        console.error(error);
        this.trailerKey = null;
      }
    },
    closeModal() {
      this.detailItem = null;
      this.trailerKey = null;
    },
    changeTab(tab) {
      if (this.tab !== tab) {
        this.tab = tab;
        this.page = 1;
        this.searchText = "";
        this.fetchItems();
        this.detailItem = null;
        this.trailerKey = null;
      }
    },
    changePage(page) {
      if (page >= 1 && page <= this.totalPages && page !== this.page) {
        this.page = page;
      }
    },
    changeSearchPage(page) {
      if (page >= 1 && page <= this.searchPageCount && page !== this.searchPage) {
        this.searchPage = page;
        this.runSearch();
      }
    },
    getPosterUrl(path) {
      return path
        ? "https://image.tmdb.org/t/p/w500" + path
        : "https://via.placeholder.com/350x520?text=No+Image";
    },
    getTitle(item) {
      return this.tab === "movie" ? item.title : item.name;
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
  padding: 0 16px;
  height: 64px;
  background: #1b375c;
  color: white;
  font-weight: 500;
  gap: 24px;
  flex-wrap: wrap;
  font-size: 1.05rem;
}
.tmdb-logo {
  font-size: 1.25rem;
  font-weight: bold;
  display: flex;
  align-items: center;
  gap: 7px;
}
.logo-dot {
  width: 16px;
  height: 8px;
  background: #39c5a5;
  border-radius: 6px;
}
.tmdb-navbar a {
  color: white;
  text-decoration: none;
  font-weight: 600;
  padding: 6px 14px 5px 14px;
  border-radius: 7px;
  transition: background-color 0.2s;
}
.tmdb-navbar a:hover,
.tmdb-navbar a.active {
  background: rgba(58, 197, 165, 0.3);
  color: #39c5a5;
}

.main-section {
  max-width: 1198px;
  margin: 32px auto 12px;
  width: 100%;
}
.top-bar {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
  margin-bottom: 28px;
}
.sort-bar {
  display: flex;
  gap: 12px;
}
.sort-bar button {
  cursor: pointer;
  border: none;
  border-radius: 8px;
  background-color: #39c5a5;
  color: white;
  padding: 8px 20px;
  font-weight: 600;
  font-size: 1rem;
  transition: background-color 0.3s;
}
.sort-bar button:hover {
  background-color: #173c66;
}
.search-input {
  flex-grow: 1;
  min-width: 180px;
  max-width: 320px;
  font-size: 1rem;
  padding: 10px 20px;
  border: 2px solid #3bc1a9;
  border-radius: 8px;
}

.movies-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(245px, 1fr));
  gap: 36px 22px;
  justify-items: center;
}
.movie-card {
  cursor: pointer;
  width: 245px;
  min-height: 430px;
  border-radius: 15px;
  box-shadow: 0 20px 40px rgb(0 0 0 / 6%);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  margin: 0;
  background: white;
}
.poster-wrap {
  height: 360px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.poster {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.card-info {
  background-color: #3b436b;
  height: 70px;
  padding: 0 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 0 0 15px 15px;
}
.movie-title {
  color: white;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 65%;
  font-size: 1rem;
}
.movie-rating {
  background-color: #39c5a5;
  padding: 6px 14px;
  border-radius: 15px;
  color: #285a80;
  font-weight: 700;
  font-size: 1rem;
}

.pagination-bar {
  margin-top: 30px;
  margin-bottom: 48px;
  display: flex;
  justify-content: center;
  gap: 10px;
  flex-wrap: wrap;
}
.page-btn {
  border: none;
  background-color: #e3e8ef;
  color: #3f4e73;
  padding: 8px 16px;
  font-weight: 600;
  border-radius: 7px;
  cursor: pointer;
  transition: all 0.3s ease;
}
.page-btn:hover {
  background-color: #3bc1a9;
  color: white;
}
.page-btn:disabled {
  opacity: 0.5;
  cursor: default;
}
.page-btn.active {
  background-color: #3bc1a9;
  color: white;
}

/* Modal */
.movie-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(36, 45, 92, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 300;
}

.modal-content {
  position: relative;
  background-color: white;
  border-radius: 20px;
  width: 900px;
  max-width: 90vw;
  height: 550px;
  max-height: 90vh;
  display: flex;
  overflow: hidden;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  font-size: 2rem;
  font-weight: 700;
  cursor: pointer;
  color: #777a8c;
}

.close-btn:hover {
  color: #3bc1a9;
}

.modal-left {
  flex: 1;
  background-color: #f0f8ff;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  border-radius: 20px 0 0 20px;
  box-shadow: 10px 0 36px rgba(199, 212, 233, 0.5);
}

.poster-large {
  max-width: 280px;
  max-height: 410px;
  border-radius: 15px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.3);
  object-fit: contain;
  background: #cfd7eb;
}

.modal-right {
  flex: 2;
  padding: 30px 40px;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  justify-content: flex-start;
}

.movie-info {
  flex: 0 0 auto;
}

.movie-info h2 {
  font-size: 2rem;
  color: #16b89b;
  margin-bottom: 15px;
  font-weight: 700;
}

.score,
.date {
  font-size: 1.2rem;
  color: #3bc1a9;
  margin-bottom: 10px;
  font-weight: 600;
}

.overview {
  font-size: 1rem;
  line-height: 1.6;
  color: #555a6d;
  white-space: pre-wrap;
  margin-bottom: 20px;
}

.trailer-section {
  flex: 1;
}

.trailer-section h3 {
  font-size: 1.7rem;
  margin-bottom: 15px;
  color: #1e2f5a;
  font-weight: 700;
}

.trailer-video {
  width: 100%;
  max-width: 600px;
  height: 350px;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.trailer-video iframe {
  width: 100%;
  height: 100%;
  border: none;
}

/* Responsive */

@media (max-width: 900px) {
  .modal-content {
    flex-direction: column;
    height: auto;
    max-height: 85vh;
  }

  .modal-left {
    width: 100%;
    border-radius: 20px 20px 0 0;
    box-shadow: none;
    max-height: 350px;
  }

  .poster-large {
    max-width: 180px;
    max-height: 350px;
    margin-bottom: 20px;
  }

  .modal-right {
    width: 100%;
    padding: 20px 25px;
    max-height: 350px;
    overflow-y: auto;
  }

  .trailer-video {
    height: 200px;
    max-width: 100%;
  }
}
</style>
