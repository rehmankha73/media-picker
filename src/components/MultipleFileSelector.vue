<template>
  <div class="drop display-inline align-center mt-6" @dragover.prevent @drop="onDrop">
    <label class="btn display-inline mt-2">
      {{ label }}
      <input type="file" @change="onChange" multiple>
    </label>

  </div>

  <div v-if="enableType" class="mt-4">
    <label for="fileType" v-if="enableType">Type of files:</label>
    <input v-model="fileType" type="text" id="fileType"
           class="shadow appearance-none border rounded w-1/2 py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
           v-if="enableType">
  </div>

  {{ files }}

  Uploaded Files: {{ no_of_files ? no_of_files : 0 }}
  <div v-for="(file,index) in files">
    <p  class="mt-4">
      Name: {{ files_data[index].name }}<br>
      Extensions: {{ files_data[index].type }}<br>
    </p>

    <button class="btn mt-2" @click="removeFile(index)">REMOVE</button><br>

    <img v-if="fileType === 'image/*'" :src="file" alt="" class="img" />

    <video v-if="fileType === 'video/*' && this.files_data[index].type === 'video/mp4'" width="320" height="240" class="mx-auto block" controls>
      <source :src="file" type="video/mp4">
    </video>


  </div>


</template>

<script>
export default {
  name: "FileSelector",
  data() {
    return {
      no_of_files: "",
      files_data: [],
      files: [],
      fileType: "image/*"
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
    }
  },
  watch: {
    files: {
      handler(newVal) {
        this.no_of_files = this.files.length
        this.$emit("getUploadedFiles", this.files);
      }, deep: true
    }
  },
  methods: {
    onDrop: function(e) {
      e.stopPropagation();
      e.preventDefault();
      let _files = e.dataTransfer.files;
      for (let f = 0; f < _files.length; f++) {
        this.files_data[f] = _files[f];
      }
      this.createFile(_files);
    },

    onChange(e) {
      let _files = e.target.files;
      for (let f = 0; f < _files.length; f++) {
        this.files_data[f] = _files[f];
      }

      this.createFile(_files);
    },

    createFile(files) {
      for (let f = 0; f < files.length; f++) {
        if (!files[f].type.match(this.fileType)) {
          alert("Please Select Files of Same types.");
          return;
        }
      }

      for (let i = 0; i < files.length; i++) {
        let reader = new FileReader();
        let vm = this;

        reader.onload = function(e) {
          vm.files[i] = e.target.result;
        };

        reader.readAsDataURL(files[i]);
      }

    },

    removeFile(_index) {
      this.files.splice(_index,1);
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

