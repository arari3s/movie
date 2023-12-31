<template>
  <div>
    <div v-if="movie">
      <div
        class="bg-slate-600 bg-cover bg-center bg-no-repeat p-8 bg-blend-multiply"
        :style="{
          backgroundImage: getBackdropUrl(movie.backdrop_path),
        }"
      >
        <div class="mx-auto max-w-screen-lg px-4 py-8">
          <div class="flex-none items-center lg:flex">
            <img
              :src="getImageUrl(movie.poster_path)"
              alt="Movie Poster"
              class="mb-4 mr-8 w-80 rounded-lg shadow-md lg:mb-0"
            />
            <div>
              <h1 class="mb-2 text-3xl font-bold text-slate-200">
                {{ movie.title }}
                <span class="font-light">
                  ({{
                    new Date(movie.release_date).toLocaleString('en-us', {
                      year: 'numeric',
                    })
                  }})
                </span>
              </h1>

              <div class="mb-2 flex-none lg:flex">
                <p class="mr-1 text-sm font-light text-slate-200">
                  {{ formatDate(movie.release_date) }}
                </p>
                <p class="mr-1 text-sm font-light text-slate-200">
                  <span class="hidden font-semibold lg:inline">| </span
                  >{{ movieGenres }}
                </p>
                <p class="text-sm font-light text-slate-200">
                  <span class="hidden font-semibold lg:inline">| </span
                  >{{ formatDuration(movie.runtime) }}
                </p>
              </div>

              <div class="mb-2 flex-none items-start lg:flex">
                <p class="mb-4 mr-8 font-semibold text-slate-200">
                  {{ getVotePercentage(movie.vote_average) }}
                  <span class="font-light">User Score</span>
                </p>
                <!-- Tampilkan tombol "View Trailer" -->
                <button
                  @click="openTrailerModal"
                  class="flex items-center bg-transparent fill-slate-200 font-semibold text-slate-200 hover:fill-slate-500 hover:text-slate-500"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="20"
                    height="20"
                    viewBox="0 0 24 24"
                    class="mr-2"
                  >
                    <path d="M3 22v-20l18 10-18 10z" />
                  </svg>
                  View Trailer
                </button>
              </div>

              <p class="mb-2 font-light italic text-slate-200">
                {{ movie.tagline }}
              </p>
              <p class="mb-2 text-xl font-semibold text-slate-200">Overview</p>
              <p class="text-slate-200">{{ movie.overview }}</p>
            </div>
          </div>
        </div>
      </div>

      <div class="mx-auto max-w-screen-lg px-4 py-8">
        <h2 class="text-xl font-semibold text-slate-800">Top Billed Cast</h2>
        <div class="mb-10 flex flex-nowrap overflow-x-auto">
          <div
            v-for="cast in limitedCast"
            :key="cast.id"
            class="flex-none pb-4 pr-4 pt-4"
          >
            <div class="w-32 bg-white">
              <img
                v-if="cast.profile_path"
                :src="getProfileImageUrl(cast.profile_path)"
                :alt="cast.name"
                class="w-32 rounded-t-lg"
              />

              <div class="pb-6 pt-2">
                <h3 class="text-sm font-semibold tracking-tight text-slate-800">
                  {{ cast.name }}
                </h3>
                <p class="text-sm font-light text-slate-600">
                  {{ cast.character }}
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Tampilkan trailer dalam modal / popup -->
      <div
        v-if="showTrailerModal"
        class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50"
      >
        <!-- Modal content -->
        <div
          class="relative h-auto w-full max-w-screen-2xl rounded-lg bg-slate-800"
        >
          <!-- Modal header -->
          <div class="flex items-start justify-between rounded-t p-4">
            <h3 class="text-xl font-semibold text-slate-200">Play Trailer</h3>
            <button
              @click="closeTrailerModal"
              class="ml-auto inline-flex h-8 w-8 items-center justify-center rounded-lg bg-transparent text-sm text-slate-400 hover:bg-slate-600 hover:text-white"
            >
              <svg
                class="h-3 w-3"
                aria-hidden="true"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 14 14"
              >
                <path
                  stroke="currentColor"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"
                />
              </svg>
              <span class="sr-only">Close modal</span>
            </button>
          </div>
          <!-- Modal body -->
          <div class="space-y-6 p-3">
            <iframe
              width="100%"
              height="825"
              :src="getTrailerUrl(movie.trailer.key)"
              frameborder="0"
              allowfullscreen
            ></iframe>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Loading...</p>
    </div>
  </div>
</template>

<script>
export default {
  layout: 'dashboard',
  data() {
    return {
      showTrailerModal: false,
    }
  },
  async asyncData({ params, $axios }) {
    try {
      const movieId = params.id
      const apiKey = process.env.TMDB_API_KEY
      // Pemanggilan API berdasarkan ID film dan Ambil data pemeran unggulan (cast) dari API
      const [movieResponse, creditsResponse, videosResponse] =
        await Promise.all([
          $axios.get(`/movie/${movieId}?api_key=${apiKey}`),
          $axios.get(`/movie/${movieId}/credits?api_key=${apiKey}`),
          $axios.get(`/movie/${movieId}/videos?api_key=${apiKey}`),
        ])

      const trailer = videosResponse.data.results.find(
        (video) => video.type === 'Trailer'
      )

      return {
        movie: {
          ...movieResponse.data,
          credits: creditsResponse.data,
          trailer,
        },
      }
    } catch (error) {
      console.error('Error:', error)
      return {
        movie: null,
      }
    }
  },
  methods: {
    getImageUrl(posterPath) {
      if (!posterPath) {
        return 'https://via.placeholder.com/300x450?text=No+Poster'
      }
      return `https://image.tmdb.org/t/p/w300${posterPath}`
    },
    getBackdropUrl(backdropPath) {
      if (!backdropPath) {
        return 'https://via.placeholder.com/1080x500?text=No+Poster'
      }
      return `url('https://image.tmdb.org/t/p/original${backdropPath}')`
    },
    getProfileImageUrl(profilePath) {
      return `https://image.tmdb.org/t/p/w276_and_h350_face${profilePath}`
    },
    getVotePercentage(voteAverage) {
      return (voteAverage * 10).toFixed(0) + '%'
    },
    formatDate(dateString) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      const date = new Date(dateString)
      // format indonesia
      // return date.toLocaleDateString('id-ID', options)
      // format english
      return date.toLocaleDateString('en-us', options)
    },
    formatDuration(minutes) {
      const hours = Math.floor(minutes / 60)
      const remainingMinutes = minutes % 60
      return `${hours}h ${remainingMinutes}m`
    },
    getTrailerUrl(key) {
      return `https://www.youtube.com/embed/${key}`
    },
    openTrailerModal() {
      this.showTrailerModal = true
    },
    closeTrailerModal() {
      this.showTrailerModal = false
    },
  },
  computed: {
    movieGenres() {
      if (this.movie.genres) {
        return this.movie.genres.map((genre) => genre.name).join(', ')
      }
      return 'Genre tidak tersedia'
    },
    limitedCast() {
      // Batasi jumlah pemeran yang ditampilkan, misalnya 10
      return this.movie.credits.cast.slice(0, 10)
    },
  },
}
</script>
