<template>
  <div class="drop display-inline align-center mt-6" @dragover.prevent @drop="onDrop">
    <label v-if="!file" class="btn display-inline mt-2">
      {{ label }}
      <input type="file" name="image" @change="onChange">
    </label>
    <label v-else class="hidden display-inline align-center" :class="{ 'image': true }">
      <div>
        <br>
        <button class="btn mt-2" @click="removeFile">REMOVE</button>
      </div>
    </label>
  </div>

  <div v-if="enableType" class="mt-4">
    <label for="fileType" v-if="enableType">Type of files:</label>
    <input v-model="fileType" type="text" id="fileType"
           class="shadow appearance-none border rounded w-1/2 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
           v-if="enableType">
  </div>

  <p class="mt-4">
    Name: {{ this.file_data.name }}<br>
    Extensions: {{ this.file_data.type }}<br>
    Uploaded File: {{ no_of_files ? no_of_files : 0 }}
  </p>

  <img v-if="fileType === 'image/*'" :src="file" alt="" class="img" />

  <video v-if="fileType === 'video/*' && file_data.type === 'video/mp4'" width="320" height="240" class="mx-auto block" controls>
    <source :src="file" type="video/mp4">
  </video>

</template>

<script>
export default {
  name: "FileSelector",
  data() {
    return {
      file_data: "",
      file: "",
      fileType: "image/*",
      no_of_files: "",
      file_extension: ""
    };
  },
  props: {
    label: {
      type: String,
      default: "SELECT OR DROP AN IMAGE"
    },
    enableType: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    file: {
      handler(newVal) {
        const obj = {
          file: newVal,
          file_data: this.file_data
        };
        this.no_of_files = 1;
        this.$emit("getUploadedFile", obj);
      }, deep: true
    }
  },
  methods: {
    onDrop: function(e) {
      e.stopPropagation();
      e.preventDefault();
      let files = e.dataTransfer.files;
      this.file_data = files[0];
      this.createFile(files[0]);
    },

    onChange(e) {
      let files = e.target.files;
      console.log(files);
      this.file_data = files[0];
      this.createFile(files[0]);

    },

    createFile(file) {
      if (!file.type.match(this.fileType)) {
        alert("Please Select File of appropriate type, default:images");
        return;
      }

      let reader = new FileReader();
      let vm = this;

      reader.onload = function(e) {
        vm.file = e.target.result;
      };
      reader.readAsDataURL(file);
    },

    removeFile() {
      this.file = "";
    }
  }
};
</script>

<style>
* {
  font-family: 'Arial';
  font-size: 12px;
}

*,
*:after,
*:before {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  -webkit-touch-callout: none;
}

html, body {
  height: 100%;
  text-align: center;
}

.btn {
  background-color: #d3394c;
  border: 0;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  font-weight: bold;
  padding: 15px 35px;
  position: relative;
}

.btn:hover {
  background-color: #722040;
}

input[type="file"] {
  position: absolute;
  opacity: 0;
  z-index: -1;
}

.align-center {
  text-align: center;
}

.helper {
  height: 100%;
  display: inline-block;
  vertical-align: middle;
  width: 0;
}

.hidden {
  display: none !important;
}

.hidden.image {
  display: inline-block !important;
}

.display-inline {
  display: inline-block;
  vertical-align: middle;
}

.img {
  border: 1px solid #f6f6f6;
  display: inline-block;
  height: auto;
  max-height: 80%;
  max-width: 80%;
  width: auto;
}

.drop {
  background-color: #f2f2f2;
  border: 4px dashed #ccc;
  background-color: #f6f6f6;
  border-radius: 2px;
  height: 100%;
  max-height: 400px;
  max-width: 600px;
  width: 100%;
}
</style>

