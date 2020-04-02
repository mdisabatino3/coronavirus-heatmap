<template>
 <div style="position: relative;">
   <dashboard></dashboard>
 </div>
</template>

<script>
import CoronavirusDashboard from './CoronavirusDashboard';
export default {
 props: [],
 components: {
   'dashboard': CoronavirusDashboard,
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