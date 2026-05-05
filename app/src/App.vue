<template>
  <EmployeeManager />
</template>

<script>
import EmployeeManager from './components/EmployeeManager.vue'

export default {
  name: 'App',
  components: {
    EmployeeManager
  }
}
</script>

<style>
/* Reset body to allow Bootstrap to manage the base styling */
body {
  margin: 0;
  padding: 0;
  background-color: #f8f9fa; /* Light background to match Bootstrap cards */
}
</style>
