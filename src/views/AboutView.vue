<template>
  <div class="about">
    <div class="view-info-box">
      <div v-for="item in dataLocalStorage" :key="item.id" class="item-drop" :class="item.component"
        @click="showAlert(item.id)">
        {{ item.text }}
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
// import { ref } from 'vue'
// localStorage.dataTam;
let dataLocalStorage = JSON.parse(localStorage.dataTam);
console.log('Data Local Storage: ', dataLocalStorage);


const showAlert = (id: any) => {
  dataLocalStorage.forEach((element: any) => {
    if (element.id == id) {
      alert(element.props.message);
    }
  });
}
</script>
<style>

</style>
