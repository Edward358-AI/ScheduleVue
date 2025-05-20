<script setup>
import { ref, computed, reactive, watch } from 'vue'

const c = JSON.parse(window.classes);
const lzstring = window.LZString;

const htmlElement = document.documentElement
const data = ref("");
const dataMessage = ref("");

const user = reactive({
  currentPage: "planner",
  schedule: { ninth: [], tenth: [], eleventh: [], twelfth: [] },
  theme: "light"
})

user.theme = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).theme : user.theme
user.schedule = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).schedule : user.schedule
user.currentPage = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).currentPage : user.currentPage

htmlElement.setAttribute("data-bs-theme", user.theme)

const catalogSearch = ref("")

const filteredClasses = computed(() => {
  if (catalogSearch.value) {
    let filteredClasses = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].name.toLowerCase().includes(catalogSearch.value.toLowerCase())) filteredClasses[course] = c[course]
    }
    return filteredClasses
  } else {
    return reactive(c)
  }

})

function exportData() {
  data.value = lzstring.compressToBase64(JSON.stringify(user))
  dataMessage.value = "Successfully exported data. Copy and paste the text below."
  setTimeout(() => {
    dataMessage.value = "";
  }, 3000)
}

function importData() {
  if (data.value.trim() && data.value.length % 4 === 0 && lzstring.decompressFromBase64(data.value)) {
    if (confirm("Are you sure you want to import this data?")) {
      localStorage.setItem("user", lzstring.decompressFromBase64(data.value))
      user.theme = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).theme : user.theme
      user.schedule = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).schedule : user.schedule
      user.currentPage = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")).currentPage : user.currentPage
      data.value = ""
      dataMessage.value = "Successfully imported data."
      setTimeout(() => {
        dataMessage.value = "";
      }, 3000)
    }
  } else {
    dataMessage.value = "An error occured during import. Please make sure there is no whitespace/newlines anywhere and the correct save encoding has been used."
    setTimeout(() => {
      dataMessage.value = "";
    }, 3000)
  }
}

watch(user, () => {
  localStorage.setItem("user", JSON.stringify(user))
  htmlElement.setAttribute("data-bs-theme", user.theme)
})

</script>

<template>
  <nav class="navbar navbar-expand-lg"
    style="background-color: var(--bs-content-bg); border-bottom: var(--bs-border-width) solid var(--bs-content-border-color);">
    <div class="container-fluid">
      <a class="navbar-brand">
        <img src="./assets/favicon.png" alt="Logo" width="24" height="24" class="d-inline-block align-text-top">
        ScheduleVue
      </a>
      <div class="collapse navbar-collapse" id="navbar-collapse-1">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item">
            <a class="nav-link" :class="user.currentPage == 'planner' ? 'active' : ''" href="#"
              @click="user.currentPage = 'planner'">Course Planner</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" :class="user.currentPage == 'catalog' ? 'active' : ''" href="#"
              @click="user.currentPage = 'catalog'">Course Catalog</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" :class="user.currentPage == 'settings' ? 'active' : ''" href="#"
              @click="user.currentPage = 'settings'">Settings</a>
          </li>
        </ul>
        <ul class="navbar-nav d-flex">
          <li class="nav-item">
            <a class="nav-link" target="_blank" href="https://github.com/Edward358-AI/ScheduleVue">Github</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" target="_blank"
              href="https://github.com/Edward358-AI/ScheduleVue/blob/main/LICENSE">License</a>
          </li>
        </ul>
      </div>
    </div>
  </nav>

  <div class="container-fluid mt-5" v-show="user.currentPage == 'settings'">
    <h4 class="mb-5">Settings</h4>
    <label for="themeselect" class="form-label">
      Theme:
      <select v-model="user.theme" class="form-select">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </label><br><br>
    <span>Import/Export Course Data: </span><br>
    <button class="btn btn-secondary me-3" @click="exportData()">Export</button>
    <button class="btn btn-secondary" @click="importData()">Import</button><br><br>
    <textarea v-model="data">{{ data }}</textarea><br>
    <span>{{ dataMessage }}</span>
  </div>

  <div class="container-fluid mt-5" v-show="user.currentPage == 'planner'">
    <h4>Course Planner</h4>
  </div>

  <div class="container-fluid mt-5" v-show="user.currentPage == 'catalog'">
    <h4>Course Catalog</h4>
    <input type="text" class="form-control specific-w-300 mx-auto mb-3" id="catalogSearch" placeholder="Enter course name here..." v-model="catalogSearch">
    <div class="list-group mx-auto" style="max-width:650px">
      <a href="#" @click.prevent="className.showing = !className.showing"
        :class="(className.showing) ? 'list-group-item-secondary' : ''"
        class="list-group-item list-group-item-action py-3" v-for="className in filteredClasses">
        <span style="font-size:1rem;font-weight:bold;">{{ className.name }}</span>
        <div class="container-fluid" v-show="className.showing">
          <hr style="opacity:1">
          Credits: {{ className.credits }}<br>
          Description: {{ className.description }}<br>
          Years: {{ className.years.toString() }}<br>
          {{ (className.prerequisite) ? 'Prerequisite: ' + className.prerequisite : "" }}
        </div>
      </a>
    </div>
  </div>
</template>

<style scoped></style>
