<template>
  <div class="home">
    <img class="mx-auto block" alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Bismillah !" />
    <h1 class="text-green-500">Welcome to Vue Media Picker</h1>

    <hr>

    <file-selector
      :enableType="true"
      label="Please Select an File"
      @getUploadedFile="getFile"
    />

  </div>

</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import FileSelector from "@/components/FileSelector.vue";

export default {
  name: "HomeView",
  components: {
    HelloWorld,
    FileSelector
  },
  methods: {
    getFile(_data) {
      console.log(_data)
    }
  }
};
</script>
