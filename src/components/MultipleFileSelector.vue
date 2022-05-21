<template>
  <div class="drop-box display-inline align-center mt-6" @dragover.prevent @drop=" onDrop "
       @click="files.length === 0 ? $refs['file-input'].click() : ''"
  >
    <label v-if="files.length === 0" class="input-message mt-2">
      {{ label }}
      <input class="file-input" ref="file-input" type="file" name="image" @change="onChange" multiple>
    </label>
    <label v-else class="hidden display-inline align-center mx-4" :class="{ 'image': true }">
      <div v-for="(file, index) in files" class="d-flex justify-content-between my-2">

        <img v-if="file && fileType === 'image/*'" :src="file" :alt="file.name" width="50px" height="50px"
             class="image-preview" />

        <img v-if="file && fileType === 'application/*'" src="../assets/file_icon.jpg" :alt="file.name"
             style="width: 50px; height: 50px" />

        <img v-if="file && fileType === 'video/*'" src="../assets/video_icon.png" :alt="file.name"
             style="width: 50px; height: 50px" />

        <p class="mt-4">
          Name: {{ files_data[index].name }}<br>
          Extensions: {{ files_data[index].type }}<br>
        </p>

        <span v-if="file"
              class="mt-2 text-danger border-0 cursor-pointer" @click.stop="removeFile(index)">X</span>
      </div>
    </label>
  </div>
  <br>
  <span v-if="error" class="text-danger">{{ error }}</span>

  <div v-if="enableType" class="mt-4">
    <label for="fileType" v-if="enableType">Type of files:</label>
    <input v-model="fileType" type="text" id="fileType"
           class="shadow appearance-none border rounded w-1/2 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
           v-if="enableType">
  </div>

  {{ files }}

  Uploaded Files: {{ no_of_files ? no_of_files : 0 }}
  <div v-for="(file,index) in files">
    <p class="mt-4">
      Name: {{ files_data[index].name }}<br>
      Extensions: {{ files_data[index].type }}<br>
    </p>

    <button class="btn mt-2" @click="removeFile(index)">REMOVE</button>
    <br>

    <img v-if="fileType === 'image/*'" :src="file" alt="" class="img" />

    <video v-if="fileType === 'video/*' && this.files_data[index].type === 'video/mp4'" width="320" height="240"
           class="mx-auto block" controls>
      <source :src="file" type="video/mp4">
    </video>

  </div>


</template>

<script>
export default {
  name: "FileSelector",
  data() {
    return {
      error: "",
      no_of_files: "",
      files_data: [],
      files: [],
      fileType: "video/*"
    };
  },
  props: {
    label: {
      type: String,
      default: "SELECT OR DROP AN IMAGES/FILES"
    },
    enableType: {
      type: Boolean,
      default: false
    },
    formErrors: {
      type: String,
      default: "PLease Upload related typed documents which are allowed!"
    }
  },
  watch: {
    files: {
      handler(newVal) {
        this.no_of_files = this.files.length;
        this.$emit("getUploadedFiles", this.files);
      }, deep: true
    }
  },
  methods: {
    onDrop: function(e) {
      e.stopPropagation();
      e.preventDefault();
      let _files = e.dataTransfer.files;
      let _valid_files = [];

      for (let f = 0; f < _files.length; f++) {
        if (!_files[f].type.match(this.fileType)) {
          this.error = this.formErrors;
          alert(this.formErrors);
          break;
        }
        this.files_data[f] = _files[f];
        _valid_files.push(_files[f]);
      }

      this.createFile(_valid_files);

    },

    onChange(e) {
      let _files = e.target.files;
      let _valid_files = [];
      for (let f = 0; f < _files.length; f++) {
        if (!_files[f].type.match(this.fileType)) {
          this.error = this.formErrors;
          alert(this.formErrors);
          break;
        }
        this.files_data[f] = _files[f];
        _valid_files.push(_files[f]);
      }

      this.createFile(_valid_files);
    },

    createFile(files) {

      for (let i = 0; i < files.length; i++) {
        let reader = new FileReader();
        let vm = this;

        reader.readAsDataURL(files[i]);

        reader.onload = function(e) {
          vm.files[i] = reader.result;
        };

      }

    },

    removeFile(_index) {
      this.files.splice(_index, 1);
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
  margin: 5px 0px;
  padding: 5px 20px;
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
  height: 500px;
  cursor: pointer;
}

.file-input {
  display: none;
}

.input-message {
  font-size: 30px;
}
</style>

