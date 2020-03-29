<template>
 <div style="position: relative;">
   <dashboard></dashboard>
 </div>
</template>

<script>
import Dashboard from './Dashboard';
export default {
 props: [],
 components: {
   'dashboard': Dashboard,
 },
 data() {
   console.log("in Content")
    return {

    }
 }
}
</script>

<style>
</style>