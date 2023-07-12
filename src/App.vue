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
    <Cropp
      ref="croppRef"
      croppfourthColor="red"
      :croppwidth="400"
      :fileOrUrl="fileval"
      :backGroundColor="true"
      @moveupCropp="moveupCropp"
    />
  </div>
  <button @click="cancenEvent">取消</button>
  <button @click="confirmEvent">确定</button>

  <button @click="confirmImage">下载</button>
  <input type="file" @change="uploadEvent" />
</template>

<script setup lang="ts">
import Cropp from "./components/croppicture/index.vue";
import { computed, ref } from "vue";

const fileval = ref("");
const uploadEvent = (file: any) => {
  croppRef.value.clearCanvas()

  fileval.value = file.target.files[0];
};

const croppRef = ref()

const moveupCropp = (even) => {
  // console.log(even);
};
const cancenEvent = (even) => {
  croppRef.value.clearCanvas()
};
const confirmEvent = async () => {
  console.log(await croppRef.value.confirmImage('confirm'));
}
</script>

<style scoped>
.dialog {
  margin-left: 300px;
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
