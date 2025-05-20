<script setup>
import { ref, computed, reactive, watch } from 'vue'
import Catalog from "./components/Catalog.vue"
import Planner from "./components/Planner.vue"
import Settings from "./components/Settings.vue"

const user = new reactive({
  currentPage: "planner",
  schedule: { ninth: [], tenth: [], eleventh: [], twelfth: [] },
  theme: "light"
})

</script>

<template>
  <nav class="navbar navbar-expand-lg"
    style="background-color: var(--bs-content-bg); border-bottom: var(--bs-border-width) solid var(--bs-content-border-color);">
    <div class="container-fluid">
      <a class="navbar-brand" >
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
  <Planner v-show="user.currentPage == 'planner'" />
  <Catalog v-show="user.currentPage == 'catalog'" />
  <Settings v-show="user.currentPage == 'settings'" />
</template>

<style scoped></style>
