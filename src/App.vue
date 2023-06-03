<template>
  <!-- <Cropp
    ref="CroppRef"
    :croppwidth="800"
    :croppheight="400"
    :croppBoxWidth="200"
    :croppBoxHeight="200"
    :fileOrUrl="fileval"
    @moveupCropp="moveupCropp"
    @confirmCropp="confirmCropp"
  /> -->
  <Cropp
    :croppwidth="410"
    :croppheight="600"
    :fileOrUrl="fileval"
    :backGroundColor="false"
    @moveupCropp="moveupCropp"
    @confirmCropp="confirmCropp"
  />
  <button @click="confirmImage">下载</button>
  <input type="file" @change="uploadEvent" />
</template>

<script setup lang="ts">
import Cropp from "./components/croppicture/index.vue";
import { computed, ref } from "vue";

const fileval = ref();
const uploadEvent = (file: any) => {
  fileval.value = file.target.files[0];
};

const moveupCropp = (even) => {
  console.log(even);
};
const confirmCropp = (even) => {
  console.log(even);
};
</script>

<style scoped></style>
