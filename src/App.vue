<script setup>
import { ref, computed, reactive, watch } from 'vue'
import pako from 'pako'

const c = reactive(JSON.parse(window.classes));
const decoder = new TextDecoder();
const encoder = new TextEncoder();
const WINDOW = window;

const htmlElement = document.documentElement
const data = ref("");
const dataMessage = ref("");

const user = reactive({
  currentPage: "planner",
  schedule: { ninth: { class1: "", class2: "", class3: "", class4: "", class5: "", class6: "", class7: "", class8: "" }, tenth: { class1: "", class2: "", class3: "", class4: "", class5: "", class6: "", class7: "", class8: "" }, eleventh: { class1: "", class2: "", class3: "", class4: "", class5: "", class6: "", class7: "", class8: "" }, twelfth: { class1: "", class2: "", class3: "", class4: "", class5: "", class6: "", class7: "", class8: "" } },
  theme: "light"
})

if(localStorage.getItem("user")) {
  user.theme = JSON.parse(localStorage.getItem("user")).theme;
}
else {
  if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
    user.theme = "dark";
  }
  else {
    user.theme = "light";
  }
}
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
    return c
  }

})

const freshman = computed(() => {
  if (catalogSearch.value) {
    let freshman = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].name.toLowerCase().includes(catalogSearch.value.toLowerCase()) && c[course].years.includes(9)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) freshman[course] = c[course]
      }
    }
    return freshman
  } else {
    let freshman = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].years.includes(9)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) freshman[course] = c[course]
      }
    }
    return freshman
  }

})

const junior = computed(() => {
  if (catalogSearch.value) {
    let junior = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].name.toLowerCase().includes(catalogSearch.value.toLowerCase()) && c[course].years.includes(11)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) junior[course] = c[course]
      }
    }
    return junior
  } else {
    let junior = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].years.includes(11)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) junior[course] = c[course]
      }
    }
    return junior
  }

})

const sophomore = computed(() => {
  if (catalogSearch.value) {
    let sophomore = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].name.toLowerCase().includes(catalogSearch.value.toLowerCase()) && c[course].years.includes(10)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) sophomore[course] = c[course]
      }
    }
    return sophomore
  } else {
    let sophomore = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].years.includes(10)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) sophomore[course] = c[course]
      }
    }
    return sophomore
  }

})

const senior = computed(() => {
  if (catalogSearch.value) {
    let senior = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].name.toLowerCase().includes(catalogSearch.value.toLowerCase()) && c[course].years.includes(12)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) senior[course] = c[course]
      }
    }
    return senior
  } else {
    let senior = reactive({})
    for (let course of Object.keys(c)) {
      if (c[course].years.includes(12)) {
        let selected = false;
        for (let grade of Object.keys(user.schedule)) {
          if (Object.values(user.schedule[grade]).includes(c[course])) selected = true;
        }
        if (!selected) senior[course] = c[course]
      }
    }
    return senior
  }

})


function bytesArrToBase64(arr) {
  const abc = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"; // base64 alphabet
  const bin = n => n.toString(2).padStart(8,0); // convert num to 8-bit binary string
  const l = arr.length
  let result = '';

  for(let i=0; i<=(l-1)/3; i++) {
    let c1 = i*3+1>=l; // case when "=" is on end
    let c2 = i*3+2>=l; // case when "=" is on end
    let chunk = bin(arr[3*i]) + bin(c1? 0:arr[3*i+1]) + bin(c2? 0:arr[3*i+2]);
    let r = chunk.match(/.{1,6}/g).map((x,j)=> j==3&&c2 ? '=' :(j==2&&c1 ? '=':abc[+('0b'+x)]));  
    result += r.join('');
  }

  return result;
}

function base64ToBytesArr(str) {
  const abc = [..."ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"]; // base64 alphabet
  let result = [];

  for(let i=0; i<str.length/4; i++) {
    let chunk = [...str.slice(4*i,4*i+4)]
    let bin = chunk.map(x=> abc.indexOf(x).toString(2).padStart(6,0)).join(''); 
    let bytes = bin.match(/.{1,8}/g).map(x=> +('0b'+x));
    result.push(...bytes.slice(0,3 - (str[4*i+2]=="=") - (str[4*i+3]=="=")));
  }
  return result;
}

// const creditsForGrad = computed(() => {
//   // math
//   for (let grade of user.schedule) {
//     if 
//   }
// })

const freshmanCredits = computed(() => {
  let sum = 0
  for (let val of Object.values(user.schedule.ninth)) {
    if (val != "") sum += val.credits
  }
  return sum
})

const sophCredits = computed(() => {
  let sum = 0
  for (let val of Object.values(user.schedule.tenth)) {
    if (val != "") sum += val.credits
  }
  return sum
})

const juniorCredits = computed(() => {
  let sum = 0
  for (let val of Object.values(user.schedule.eleventh)) {
    if (val != "") sum += val.credits
  }
  return sum
})

const seniorCredits = computed(() => {
  let sum = 0
  for (let val of Object.values(user.schedule.twelfth)) {
    if (val != "") sum += val.credits
  }
  return sum
})

function exportData() {
  const user_encoded = encoder.encode(JSON.stringify(user));
  const user_compressed = pako.deflate(user_encoded);
  data.value = bytesArrToBase64(user_compressed);
  dataMessage.value = "Successfully exported data. Copy and paste the text below."
  setTimeout(() => {
    dataMessage.value = "";
  }, 3000)
}

function importData() {
  data.value = data.value.trim();
  if (data.value.length % 4 === 0) {
    const user_compressed = base64ToBytesArr(data.value);
    const user_decompressed = pako.inflate(user_compressed);
    const user_decoded = decoder.decode(user_decompressed);
    if (confirm("Are you sure you want to import this data?")) {
      localStorage.setItem("user", user_decoded);
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
  catalogSearch.value = ""
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

  <div class="container mt-5" v-show="user.currentPage == 'settings'">
    <h4 class="mb-4">Settings</h4>
    <label for="themeselect" class="form-label">
      Theme:
      <select v-model="user.theme" class="form-select">
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </label><br><br>
    <span>Import/Export Course Data: </span><br>
    <button class="btn btn-secondary me-3" @click="exportData()">Export</button>
    <button class="btn btn-secondary me-3" @click="importData()">Import</button>
    <button class="btn btn-secondary" @click='WINDOW.localStorage.clear();WINDOW.location.reload()'>Clear Data</button><br><br>
    <textarea v-model="data">{{ data }}</textarea><br>
    <span>{{ dataMessage }}</span>
  </div>

  <div class="container-fluid container-triple-xl mt-5" v-show="user.currentPage == 'planner'">
    <h4>Course Planner</h4>
    <p>This a four-year planner designed to help you plan out your four years in high school.</p>
    <div class="row mt-5">
      <div class="col">
        <strong style="font-size:1.1rem;">9th</strong>
        <p>Credits: {{ freshmanCredits }}</p>
        <div class="dropdown dropdown-center my-3" v-for="userClass in Object.keys(user.schedule.ninth)">
          <button style="min-width: 300px" class="btn btn-secondary dropdown-toggle py-2" type="button"
            data-bs-toggle="dropdown" aria-expanded="false">
            {{ user.schedule.ninth[userClass] != "" ? user.schedule.ninth[userClass].name : "+ Select Course" }}
          </button>
          <div class="dropdown-menu course-select bg-body">
            <div style="" class="container-fluid py-2 course-selector bg-body">
              <button class="btn btn-secondary my-2 mx-auto block"
                @click.prevent="user.schedule.ninth[userClass] = ''">Clear
                Selection</button>
              <input type="text" class="form-control my-1 mx-auto block" placeholder="Enter course name..."
                v-model="catalogSearch">
            </div>
            <a href="#" @click="user.schedule.ninth[userClass] = className" class="dropdown-item"
              v-for="className in freshman" style="text-wrap:wrap;">{{ className.name }}</a>
          </div>
        </div>
      </div>
      <div class="col">
        <strong style="font-size:1.1rem;">10th</strong>
        <p>Credits: {{ sophCredits }}</p>
        <div class="dropdown dropdown-center my-3" v-for="userClass in Object.keys(user.schedule.tenth)">
          <button style="min-width: 300px" class="btn btn-secondary dropdown-toggle py-2" type="button"
            data-bs-toggle="dropdown" aria-expanded="false">
            {{ user.schedule.tenth[userClass] != "" ? user.schedule.tenth[userClass].name : "+ Select Course" }}
          </button>
          <div class="dropdown-menu course-select bg-body">
            <div style="" class="container-fluid py-2 course-selector bg-body">
              <button class="btn btn-secondary my-2 mx-auto block"
                @click.prevent="user.schedule.tenth[userClass] = ''">Clear
                Selection</button>
              <input type="text" class="form-control my-1 mx-auto block" placeholder="Enter course name..."
                v-model="catalogSearch">
            </div>
            <a href="#" @click="user.schedule.tenth[userClass] = className" class="dropdown-item"
              v-for="className in sophomore" style="text-wrap:wrap;">{{ className.name }}</a>
          </div>
        </div>
      </div>
      <div class="col">
        <strong style="font-size:1.1rem;">11th</strong>
        <p>Credits: {{ juniorCredits }}</p>
        <div class="dropdown dropdown-center my-3" v-for="userClass in Object.keys(user.schedule.eleventh)">
          <button style="min-width: 300px" class="btn btn-secondary dropdown-toggle py-2" type="button"
            data-bs-toggle="dropdown" aria-expanded="false">
            {{ user.schedule.eleventh[userClass] != "" ? user.schedule.eleventh[userClass].name : "+ Select Course" }}
          </button>
          <div class="dropdown-menu course-select bg-body">
            <div style="" class="container-fluid py-2 course-selector bg-body">
              <button class="btn btn-secondary my-2 mx-auto block"
                @click.prevent="user.schedule.eleventh[userClass] = ''">Clear
                Selection</button>
              <input type="text" class="form-control my-1 mx-auto block" placeholder="Enter course name..."
                v-model="catalogSearch">
            </div>
            <a href="#" @click="user.schedule.eleventh[userClass] = className" class="dropdown-item"
              v-for="className in junior" style="text-wrap:wrap;">{{ className.name }}</a>
          </div>
        </div>
      </div>
      <div class="col">
        <strong style="font-size:1.1rem;">12th</strong>
        <p>Credits: {{ seniorCredits }}</p>
        <div class="dropdown dropdown-center my-3" v-for="userClass in Object.keys(user.schedule.twelfth)">
          <button style="min-width: 300px" class="btn btn-secondary dropdown-toggle py-2" type="button"
            data-bs-toggle="dropdown" aria-expanded="false">
            {{ user.schedule.twelfth[userClass] != "" ? user.schedule.twelfth[userClass].name : "+ Select Course" }}
          </button>
          <div class="dropdown-menu course-select bg-body">
            <div style="" class="container-fluid py-2 course-selector bg-body">
              <button class="btn btn-secondary my-2 mx-auto block"
                @click.prevent="user.schedule.twelfth[userClass] = ''">Clear
                Selection</button>
              <input type="text" class="form-control my-1 mx-auto block" placeholder="Enter course name..."
                v-model="catalogSearch">
            </div>
            <a href="#" @click="user.schedule.twelfth[userClass] = className" class="dropdown-item"
              v-for="className in senior" style="text-wrap:wrap;">{{ className.name }}</a>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="container mt-5" v-show="user.currentPage == 'catalog'">
    <h4>Course Catalog</h4>
    <p>A comprehensive list of all the courses offered across PUSD and some neighboring schools. Click on any course to
      see a description and use the search bar below to find a specific course.</p>
    <input type="text" class="form-control specific-w-300 mx-auto mb-3" id="catalogSearch"
      placeholder="Enter course name here..." v-model="catalogSearch">
    <div class="list-group mx-auto">
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

<style scoped>
.container {
  max-width: 650px;
}

.course-select {
  width: 300px;
  max-height: 300px;
  overflow-y: scroll;
  overflow-x: hidden;
}

.course-selector {
  position: sticky;
  top: -5px;
}

.block {
  display: block;
  width: 100%;
}

.container-triple-xl {
  max-width: 1500px;
}
</style>
