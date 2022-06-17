<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <AutoComplete 
    :items="currencyItems"
    filterBy="currency"
    :addNewItem="newItem"
  />
</template>

<script>
import AutoComplete from './components/AutoComplete.vue'
import currencyItems from '@/config/dataBase'
export default {
  name: 'App',
  components: {
    AutoComplete
  },
  data() {
    return {
      currencyItems
    }
  },
  methods: {
    newItem(newValue) {
      return this.currencyItems.push({currency: newValue})
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
