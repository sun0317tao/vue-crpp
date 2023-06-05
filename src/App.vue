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
  <div class="dialog">
    lll
    <Cropp
      croppfourthColor="red"
      :croppwidth="400"
      :fileOrUrl="fileval"
      :backGroundColor="true"
      @moveupCropp="moveupCropp"
      @confirmCropp="confirmCropp"
    />
  </div>

  <button @click="confirmImage">下载</button>
  <input type="file" @change="uploadEvent" />
</template>

<script setup lang="ts">
import Cropp from "./components/croppicture/index.vue";
import { computed, ref } from "vue";

const fileval = ref('');
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

<style scoped>
.dialog {
  width: 500px;
  height: 500px;
  padding: 20px;
  /* background-color: #ccc; */
  /* position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%); */
  /* height: 2000px; */
  /* margin-left: -25%;
  margin-top: -25%; */
}
</style>
