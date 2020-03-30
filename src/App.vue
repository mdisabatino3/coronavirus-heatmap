<template>
  <div id="ancestor">
    <dashboard-content
      class="col-md-12 col-sm-12 col-xs-12 content"
      id="dashboard-content"
    ></dashboard-content>
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
</style>
