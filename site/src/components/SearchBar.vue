<script>
import exercises from '../../../dist/exercises.json'
import ExerciseInstructions from './ExerciseInstructions.vue'
import PhotoGallery from './PhotoGallery.vue'

// import bookmark icon from heroicons
import { BookmarkIcon as BookmarkIconOutline } from '@heroicons/vue/24/outline'
import { BookmarkIcon as BookmarkIconSolid } from '@heroicons/vue/24/solid'

import Fuse from 'fuse.js'

export default {
  components: {
    ExerciseInstructions,
    PhotoGallery,
    BookmarkIconOutline,
    BookmarkIconSolid
  },
  data() {
    return {
      query: '',
      exercises: exercises,
      searchResults: exercises,
      pageSize: 50,
      currentPage: 0,
      savedExercises: [],
      showSavedExercises: false,

      //filters params
      name: '',
      level: '',
      primaryMuscles: '', 
      category: '',
      equipment: '', 
      force: ''
    }
  },
  computed: {
    
    // TODO: Refactor this, it's a mess
    savedItemClasses() {
      let color = this.showSavedExercises ? 'blue' : 'gray'

      let colors = {
        [`bg-${color}-700`]: true,
        [`hover:bg-${color}-800`]: true,
        [`dark:bg-${color}-800`]: true,
        [`dark:hover:bg-${color}-700`]: true,
        [`dark:focus:ring-${color}-800`]: true,
        [`focus:ring-${color}-300`]: true
      }

      return colors
    },
    paginatedItems() {
      const startIndex = this.currentPage * this.pageSize
      const endIndex = startIndex + this.pageSize
      return this.searchResults.slice(0, endIndex)
    }
  },
  methods: {
    totalPages() {
      return Math.ceil(this.searchResults.length / this.pageSize)
    },
    saveExercise(exercise) {
      // add the exercise if it's not already in the array otherwise remove it
      if (!this.isBookedMarked(exercise)) {
        this.savedExercises.push(exercise)
      } else {
        this.savedExercises = this.savedExercises.filter((e) => e !== exercise)

        // if we ended up with no exercises then let's clear the search and reset the results
        if (this.savedExercises.length == 0) {
          this.query = ''
          this.searchResults = this.exercises
          this.showSavedExercises = false
        } else {
          this.searchResults = this.savedExercises
        }
      }
    },
    toggleSavedExercises() {
      // toggle between showing all exercises and saved exercises
      if (this.searchResults === this.savedExercises) {
        this.searchResults = this.exercises
        this.showSavedExercises = false
      } else if (this.savedExercises.length > 0) {
        this.searchResults = this.savedExercises
        this.showSavedExercises = true
      }
    },
    isBookedMarked(exercise) {
      return this.savedExercises.includes(exercise)
    },
    // isBookedMarked1() {
    //   console.log("sssass")
    // }

    query1() {
      console.log("query1 func")
      //TODO: each func or save values
      const options = {
        keys: ['id', 'name', 'level'] //, 'primaryMuscles', 'category', 'equipment', 'force']
        // keys: ['id', 'name']

      }

      this.currentPage = 0
      this.showSavedExercises = false
      console.log(this.name, "PARAM", this.level, this.name)

      if (this.name.length > 1) {
        const fuse = new Fuse(this.exercises, options)
        // this.searchResults = fuse.search({ name: newValue }).map((r) => r.item)
        this.searchResults = fuse.search({ name: this.name, level: "beginner"})//, primaryMuscles: this.primaryMuscles, category: this.category, equipment: this.equipment, force: this.force}).map((r) => r.item)

        // this.searchResults = fuse.search({ name: this.name, level: this.level})//, primaryMuscles: this.primaryMuscles, category: this.category, equipment: this.equipment, force: this.force}).map((r) => r.item)
          // ,primaryMuscles:, category:, equipment: , force:
      } else {
        this.searchResults = this.exercises
      }


      console.log(this.searchResults)
    }

 
  },
  mounted() {
    window.addEventListener('scroll', () => {
      if (window.scrollY + window.innerHeight >= document.documentElement.scrollHeight) {
        if (this.totalPages() >= this.currentPage + 1) {
          // FIXME: Add a slight delay to the endless scroll
          // as it causes repaint issues otherwise
          setTimeout(() => {
            this.currentPage = this.currentPage + 1
          }, 400)
        }
      }
    })

    // load saved exercises from local storage
    if (localStorage.getItem('savedExercises')) {
      this.savedExercises = JSON.parse(localStorage.getItem('savedExercises'))
    }
  },
  watch: {
    savedExercises: {
      handler: function (val) {
        localStorage.setItem('savedExercises', JSON.stringify(val))
      },
      deep: true
    },

    // query() {
    //   console.log("qqqq")
    //   //TODO: each func or save values
    //   const options = {
    //     keys: ['id', 'name', 'level']
    //     // , 'primaryMuscles', 'category', 'equipment', 'force']
    //     // keys: ['id', 'name']

    //   }

    //   this.currentPage = 0
    //   this.showSavedExercises = false
    //   console.log("PARAM", this.level, this.name)

    //   if (this.query.length > 1) {
    //     const fuse = new Fuse(this.exercises, options)
    //     // this.searchResults = fuse.search({ name: newValue }).map((r) => r.item)
    //     this.searchResults = fuse.search({ name: this.name, level: this.level}).map((r) => r.item)
    //       // ,primaryMuscles:, category:, equipment: , force:
    //   } else {
    //     this.searchResults = this.exercises
    //   }


    //   console.log(this.searchResults, this.level)
    // }
  }
}
</script>
<template>
  <div class="flex">
    <div class="w-full">
      <form @submit.prevent="onSubmit">
        <label
          for="default-search"
          class="mb-2 text-sm font-medium text-gray-900 sr-only dark:text-white"
          >Search</label
        >
        <div class="relative">
          <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
            <svg
              aria-hidden="true"
              class="w-5 h-5 text-gray-500 dark:text-gray-400"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
              ></path>
            </svg>
          </div>
          <input
            v-model="name"
            @change="query1"
            name="search"
            type="search"
            autofocus="autofocus"
            id="search"
            class="block w-full p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            placeholder="Search Exercises, Instructions"
            required
          />
        </div>
        <!-- filters -->
        <div class="p-4">
        <select 
            class="select p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            id="select-level"
            @change="query1"
            v-model="level">
              <option value="">Level</option>
              <option value="beginner">beginner</option>
              <option value="intermediate">intermediate</option>
              <option value="expert">expert</option>
          </select>

          <select 
            class="select p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            id="select-code"
            @change="query1"
            v-model="primaryMuscles">
              <option value="">Primary Muscles</option>
              <option value="quadriceps">quadriceps</option>
              <option value="shoulders">shoulders</option>
              <option value="abdominals">abdominals</option>
              <option value="hamstrings">hamstrings</option>
              <option value="triceps">triceps</option>
              <option value="biceps">biceps</option>
              <option value="lats">lats</option>
              <option value="middle">middle back</option>
              <option value="calves">calves</option>
              <option value="lower">lower back</option>
              <option value="forearms">forearms</option>
              <option value="glutes">glutes</option>
              <option value="traps">traps</option>
              <option value="adductors">adductors</option>
              <option value="abductors">abductors</option>
              <option value="neck">neck</option>
          </select>

          <select 
            class="select p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            id="select-code"
            @change="query1"
            v-model="category">
              <option value="">Category</option>
              <option value="strength">strength</option>
              <option value="stretching">stretching</option>
              <option value="plyometrics">plyometrics</option>
              <option value="powerlifting">powerlifting</option>
              <option value="olympic">olympic weightlifting</option>
              <option value="strongman">strongman</option>
              <option value="cardio">cardio</option>
          </select>

          <select 
            class="select p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            id="select-code"
            @change="query1"
            v-model="equipment">
              <option value="">Equipment</option>
              <option value="barbell">barbell</option>
              <option value="dumbbell">dumbbell</option>
              <option value="e-z curl bar">e-z curl bar</option>
              <option value="foam roll">foam roll</option>
              <option value="exercise ball">exercise ball</option>
              <option value="bands">bands</option>
              <option value="kettlebells">kettlebells</option>
              <option value="machine">machine</option>
              <option value="cable">cable</option>
              <option value="body only">body only </option>
              <option value="medicine ball">medicine ball</option>
              <option value="other">other</option>
          </select>

          <select 
            class="select p-4 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" 
            id="select-code"
            @change="query1"
            v-model="force">
              <option value="">Force</option>
              <option value="push">push</option>
              <option value="push">push</option>
              <option value="static">static</option>
          </select>
        </div>
      </form>
    </div>
    <div class="w-24 relative">
      <button
        type="button"
        @click.prevent="toggleSavedExercises"
        :class="savedItemClasses"
        class="text-white absolute right-2.5 bottom-2.5 focus:ring-4 focus:outline-none font-medium rounded-lg text-sm px-4 py-2"
      >
        <BookmarkIconOutline
          class="w-5 h-5 mr-2 -ml-1 text-white"
          v-if="savedExercises.length == 0"
        />
        <BookmarkIconSolid class="w-5 h-5 mr-2 -ml-1 text-white" v-if="savedExercises.length > 0" />
        <span class="sr-only">Saved</span>
        <div
          class="absolute inline-flex items-center justify-center w-6 h-6 text-xs font-bold text-white bg-red-500 border-2 border-white rounded-full -top-2 -right-2 dark:border-gray-900"
        >
          {{ savedExercises.length }}
        </div>
      </button>
    </div>
  </div>
  <div id="infinite-list">
    <div
      v-for="exercise in paginatedItems"
      v-bind:key="exercise.name"
      :class="savedItemClasses"
      class="exercise flex flex-col relative mt-4 items-center justify-between bg-white border border-gray-200 rounded-lg shadow md:flex-row md:max-w-xl dark:border-gray-700"
    >
      <div class="w-full md:h-auto md:w-60">
        <PhotoGallery :photos="exercise.images" />
      </div>
      <div class="w-96 p-4 leading-normal" :class="{ bookedmarked: isBookedMarked(exercise) }">
        <a
          href=""
          @click.prevent="saveExercise(exercise)"
          class="text-blue-500 hover:text-blue-600 dark:text-blue-400 dark:hover:text-blue-500"
        >
          <BookmarkIconOutline
            class="absolute drop-shadow top-4 right-4 md:top-2 md:right-2 w-8 md:w-5 text-gray-400 hover:text-gray-500 cursor-pointer"
            v-if="!isBookedMarked(exercise)"
          />
          <BookmarkIconSolid
            class="absolute drop-shadow top-4 right-4 md:top-2 md:right-2 w-8 md:w-5 text-gray-400 hover:text-gray-500 cursor-pointer"
            v-if="isBookedMarked(exercise)"
          />
        </a>
        <h5 class="mb-2 text-2xl font-bold tracking-tight text-gray-900 dark:text-white">
          {{ exercise.name }}
        </h5>
        <ExerciseInstructions :text="exercise.instructions" />
      </div>
    </div>
  </div>
</template>
