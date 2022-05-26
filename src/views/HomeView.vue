<template>
  <div class="home">
    <img class="mx-auto block" alt="Vue logo" src="../assets/logo.png" />
    <HelloWorld msg="Bismillah !" />
    <h1 class="text-green-500">Welcome to Vue Media Picker</h1>

<!--    <hr>-->

<!--    <h1 class="text-2xl text-green-500 my-4"> Single file selector </h1>-->
<!--    <file-selector-->
<!--      :enableType="true"-->
<!--      label="Please Select/Drag a File"-->
<!--      @getUploadedFile="getFile"-->
<!--    />-->

<!--    <hr/>-->

<!--    <h1 class="text-2xl text-rose-500 my-4"> Multiple files selector </h1>-->
<!--    <multiple-file-selector-->
<!--      :enableType="true"-->
<!--      label="Please Select/Drag & Drop Files"-->
<!--      @getUploadedFiles="getFiles"-->
<!--    />-->

    <h1> Media Selector </h1>
    <div class="mx-auto" style="width:50%">
      <media-selector
        :showAllowedTypes="true"
        allowedTypes="image/png,video/mp4,application/pdf"
        label="Accept images, videos & 3D modal"
        @getUploadedFiles="getFiles"

      />
    </div>

    <div class="mt-10">!!The End!!</div>

  </div>

</template>

<script>
import HelloWorld from "@/components/HelloWorld.vue";
import FileSelector from "@/components/FileSelector.vue";
import MultipleFileSelector from "@/components/MultipleFileSelector.vue";
import MediaSelector from "@/components/MediaSelector.vue";

export default {
  name: "HomeView",
  components: {
    HelloWorld,
    FileSelector,
    MultipleFileSelector,
    MediaSelector,
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
