<template>
  <div id="ancestor">
    <div class="container-fluid" id="app">
      <div class="row">
       <div id="sidebar" class="col-md-3 col-sm-4 col-xs-12 sidebar">
         <div id="search">
           <input
             id="location-input"
             type="text"
             ref="input"
             placeholder="Location?"
             @keyup.enter="organizeAllDetails"
           >
           <button id="search-btn" @click="buttonPressed">
             <img src="./assets/search-24px.svg" width="24" height="24">
           </button>
         </div>
       </div>
       <dashboard-content
         class="col-md-9 col-sm-8 col-xs-12 content"
         id="dashboard-content"
       ></dashboard-content>
      </div>
    </div>
  </div>
</template>

<script>
import Content from './components/Content';
export default {
  name: 'app',
  props: [],
  components: {
    'dashboard-content': Content
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods: {
    buttonPressed: function() {
      console.log("Im clicked");
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
