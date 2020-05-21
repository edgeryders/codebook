<template>
  <div class="main">
    <h1>{{name}}</h1>
    <input type="text" v-model="searchQuery" placeholder="Search for codes..." /> <span>with at least </span> <input type="text" v-model="occurencesFilter" class="number" placeholder="" /> <span>occurrences </span>
    <p v-if="apiData === ''">Loading...</p>
    <div class="data" v-else>
      <div v-for="dataItem in filteredData" :key="dataItem.id">
        <div v-if="dataItem.annotations_count > 0 && Number(dataItem.annotations_count) >= Number(occurrences)">
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
  .number {
    width: 20px;
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
  span {
    font-style: italic;
    font-weight: normal;
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
      occurencesFilter: 2,
      apiData: ""
    }
  },
  mounted() {
    fetch(this.apiEndpoint, {headers: {Accept: 'application/json'}, method: 'GET'}).then(data => data.json()).then(data => {
      this.apiData = [];
      data.forEach(element => {
        let term = element;
        if(element.name.slice(0,3) != '(A)') {
          if(element.name.slice(0,3) == '(C)') {
            term.name = term.name.slice(3)
            term.ancestry = 'character'
            term.name_with_path = 'character'
          }
          term.name = term.name.charAt(0).toUpperCase() + term.name.slice(1)
          if (term.description) {
            term.description = term.description.charAt(0).toUpperCase() + term.description.slice(1)
          }
          this.apiData.push(term)
        }
      })
    });
    document.title = this.name;
  },
  methods: {
    description(dataItem) {
      return ((dataItem.description) ? dataItem.description : '' )
    },
    shouldShow(dataItem) {
      return (this.description(dataItem).toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1) || 
        (dataItem.name.toLowerCase().indexOf(this.searchQuery.toLowerCase()) !== -1);
    }
  },
  computed: {
    occurrences() {
      return this.occurencesFilter;
    },
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