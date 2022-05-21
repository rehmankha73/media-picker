<template>
  <div class="home">
    <img class="mx-auto block" alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Bismillah !" />
    <h1 class="text-green-500">Welcome to Vue Media Picker</h1>

    <hr>

    <h1 class="text-2xl text-rose-500 my-4"> Single file selector </h1>
    <file-selector
      :enableType="true"
      label="Please Select an File"
      @getUploadedFile="getFile"
    />

    <hr/>

    <h1 class="text-2xl text-rose-500 my-4"> Multiple files selector </h1>
    <multiple-file-selector
      :enableType="true"
      label="Please Select Files"
      @getUploadedFiles="getFiles"
    />

    <div class="mt-10">!!The End!!</div>

  </div>

</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import FileSelector from "@/components/FileSelector.vue";
import MultipleFileSelector from "@/components/MultipleFileSelector.vue";

export default {
  name: "HomeView",
  components: {
    HelloWorld,
    FileSelector,
    MultipleFileSelector
  },
  methods: {
    getFile(_data) {
      console.log(_data);
    },
    getFiles(_data) {
      console.log(_data, "data from home view");
    }
  }
};
</script>
