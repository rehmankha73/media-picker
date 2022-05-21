<template>
  <div class="drop-box display-inline align-center mt-6" @dragover.prevent @drop="onDrop"
       @click="$refs['file-input'].click()"
  >
    <label v-if="!file" class="input-message mt-2">
      {{ label }}
      <input class="file-input" ref="file-input" type="file" name="image" @change="onChange">
    </label>
    <label v-else class="hidden display-inline align-center" :class="{ 'image': true }">
      <div>
        <img v-if="file && fileType === 'image/*'" :src="file" :alt="file.name" width="50px" height="50px" class="image-preview" />
        <br>
        <button class="btn mt-2" @click.stop="removeFile">REMOVE</button>
      </div>
    </label>
  </div>
  <br>
  <span v-if="error" class="text-danger">{{ error }}</span>

  <div v-if="enableType" class="mt-4">
    <label for="fileType" v-if="enableType">Type of files:</label>
    <input v-model="fileType" type="text" id="fileType"
           class="w-1/2 py-2 px-3 "
           v-if="enableType">
  </div>

  <p class="mt-4">
    <span v-if="this.file_data.name">Name: {{ this.file_data.name }}<br></span>
    <span v-if="this.file_data.type">Extensions: {{ this.file_data.type }}<br></span>
    Uploaded File: {{ no_of_files ? no_of_files : 0 }}
  </p>

  <img v-if="fileType === 'image/*'" :src="file" :alt="file.name" />

  <video id="video-preview" v-if="fileType === 'video/*' && file_data.type === 'video/mp4'"  controls :src="file"/>


</template>

<script>
export default {
  name: "FileSelector",
  data() {
    return {
      file_data: "",
      file: "",
      fileType: "image/*",
      error: "",
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
    },
    formErrors:{
      type: String,
      default: "Please Select File of appropriate type, default:images",
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

      // Validations errors
      if (!files[0].type.match(this.fileType)) {
        this.error = this.formErrors;
        alert(this.formErrors);
        return;
      }

      this.file_data = files[0];
      this.createFile(files[0]);
    },

    onChange(e) {
      let files = e.target.files;

      // Validations errors
      if (!files[0].type.match(this.fileType)) {
        this.error = this.formErrors;
        alert(this.formErrors);
        return;
      }

      this.file_data = files[0];
      this.createFile(files[0]);

    },

    createFile(file) {

      let reader = new FileReader();
      let vm = this;

      reader.readAsDataURL(file);

      reader.onload = function(e) {
        vm.file = reader.result;
      };

    },

    removeFile() {
      this.file = "";
    }
  }
};
</script>

<style>
* {
  font-family: 'Arial', serif;
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
  margin-top: 2px;
  padding: 15px 30px;
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

.image-preview {
  border: 1px solid #f6f6f6;
  display: inline-block;
  height: auto;
  max-height: 30%;
  max-width: 30%;
  width: auto;
}

.drop-box {
  background-color: #f2f2f2;
  border: 4px dashed #ccc;
  border-radius: 2px;
  width: 600px;
  height: 400px;
  cursor: pointer;
}

.file-input {
  display: none;
}

.input-message {
  font-size: 30px;
}
</style>

