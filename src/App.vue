<template>
  <div class="main">
    <h1>{{name}}</h1>
    <input type="text" v-model="searchQuery" placeholder="Search for codes..." />
    <p v-if="apiData === ''">Loading...</p>
    <div class="data" v-else>
      <div v-for="dataItem in filteredData" :key="dataItem.id">
        <div v-if="dataItem.annotations_count > 0">
          <h2>{{dataItem.name}} ({{dataItem.annotations_count}})</h2>
          <p class="smaller" v-if="dataItem.ancestry !== null">{{dataItem.name_with_path}}</p>
          <p>{{dataItem.description}}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
  $dull: #8a8a8a;
  * {
    font-family: "Times New Roman"
  }
  .smaller {
    font-size: 10px;
  }
  div.main {
    width: 80%;
    margin: auto;
    max-width: 700px;
  }
  @media screen and (max-width: 400px) {
    div.main {
      width: 98%;
    }
  }
  input {
    border-width: 0 0 1px 0;
    border-color: $dull;
    font-style: italic;
    color: $dull;
    margin-bottom: 20px;
    font-size: 16px;
  }
  input:focus {
    outline: 0;
  }
  h2 {
    font-size: 16px;
    margin-bottom: 5px;
    font-weight: normal;
  }
  h1 {
    font-weight: normal;
    margin-top: 30px;
  }
  p {
    margin-top: 2px;
    color: $dull;
  }
</style>

<script>
import CONFIG from "./CONFIGFILE.js"
export default {
  data() {
    return {
      ...CONFIG,
      searchQuery: "",
      apiData: ""
    }
  },
  mounted() {
    fetch(this.apiEndpoint).then(data => data.json()).then(data => {
      this.apiData = data;
    });
    document.title = this.name;
  },
  methods: {
    shouldShow(dataItem) {
      return (dataItem.description.toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1) || 
        (dataItem.name.toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1);
    }
  },
  computed: {
    filteredData() {
      return this.apiData.filter((dataItem) => this.shouldShow(dataItem)).sort((a,b) => {
        if (a.name.trim().toLowerCase() > b.name.trim().toLowerCase()) {
          return 1;
        } else if (b.name.trim().toLowerCase() > a.name.trim().toLowerCase()) {
          return -1;
        } else {
          return 0;
        }
      })
    }
  }
  
}
</script>