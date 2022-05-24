<template>
  <div class="drop-box display-inline align-center mt-6" @dragover.prevent @drop="onDrop"
       @click="files.length === 0 ? $refs['file-input'].click() : ''"
  >
    <label v-if="files.length === 0" class="input-message mt-2">
      {{ label }}
      <input class="file-input" ref="file-input" type="file" name="image" @change="onChange" multiple>
    </label>
    <label v-else class="display-inline align-center mx-4">
      <div v-for="(file, index) in files" class="d-flex justify-content-between my-2">

        <img v-if="file && fileType === 'image/*'" :src="file" :alt="file.name"
             @click="openModal(file, 'image')"
             class="image-preview cursor-pointer" />

        <embed v-else-if="file && fileType === 'application/*'" :src="file"
               class="d-block cursor-pointer" style="width: 100px; height: 100px" />

        <img v-else-if="file && fileType === 'video/*'" :src="video_thumbnails[index]" :alt="file.name"
             @click="openModal(file, 'video')"
             class="d-block cursor-pointer" style="width: 100px; height: 100px" />

        <p class="mt-4 d-block">
          Name: {{ files_data[index].name }}<br>
          Extensions: {{ files_data[index].type }}<br>
        </p>

        <span v-if="file" class="mt-2 text-danger border-0 cursor-pointer d-block"
              @click.stop="removeFile(index)">X</span>
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

  Uploaded Files: {{ no_of_files ? no_of_files : 0 }}
  <div v-for="(file,index) in files">
    <p class="mt-4">
      Name: {{ files_data[index].name }}<br>
      Extensions: {{ files_data[index].type }}<br>
    </p>
  </div>

  <div id="myModal" class="modal">
    <span class="close cursor" @click="closeModal()">&times;</span>
    <div class="modal-content">
      <div v-if="fileType === 'image/*'" id="image-slide" style="display: none;background: black">
        <img src="../assets/logo.png" id="modal-image" alt="image"
             style="width: 500px; height: 500px;background: black">
      </div>

      <div v-if="fileType === 'video/*'" id="video-slide" style="display: none; background: black">
        <video id="modal-video"
               style="width: 500px; height: 500px; background: black"
               class="mx-auto block" controls>
          <source src="" type="video/mp4" id="modal-video-source">
        </video>
      </div>
    </div>
  </div>


</template>

<script>

export default {
  name: "FileSelector",
  data() {
    return {
      video_thumbnails: [],
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
        if (newVal.length === 0) {
          this.video_thumbnails = [];
        }
        this.no_of_files = this.files.length;
        this.$emit("getUploadedFiles", this.files);
      }, deep: true
    }
  },
  methods: {
    openModal(_file, _type) {
      let image_slide = document.getElementById("image-slide");
      let modal_image = document.getElementById("modal-image");
      let video_slide = document.getElementById("video-slide");
      let modal_video = document.getElementById("modal-video");
      let modal_video_source = document.getElementById("modal-video-source");

      if (_type && _type === "image") {
        image_slide.style.display = "block";
        modal_image.src = _file;
      }

      if (_type && _type === "video") {
        video_slide.style.display = "block";

        modal_video_source.setAttribute("src", _file);

        modal_video.load();
      }
      document.getElementById("myModal").style.display = "block";
    },

    closeModal() {

      document.getElementById("myModal").style.display = "none";
    },

    async generateThumbnail(item) {
      console.log(item);
      console.log("thumbnail");
      const binaryData = [];
      binaryData.push(item);
      const canvas = document.createElement("canvas");
      const context = canvas.getContext("2d");
      const video = document.createElement("video");
      video.setAttribute("src", URL.createObjectURL(new Blob(binaryData)));
      video.load();
      let thumbnail = await new Promise((resolve) => {
        video.onloadedmetadata = async () => {
          console.log("in onload");
          canvas.width = video.videoWidth;
          canvas.height = video.videoHeight;
          video.currentTime = video.duration / 2;
          await video.play();
          context.drawImage(video, 0, 0);
          video.pause();
          const blob = await new Promise((resolve) => {
            return canvas.toBlob(function(blob) {
              resolve(URL.createObjectURL(blob));
            });

            // return canvas.toDataURL();
          });
          console.log(blob);
          resolve(blob);
        };
      });

      console.log(thumbnail, "thumb");

      this.video_thumbnails.push(thumbnail);
      // return thumbnail;
    },

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

    async createFile(_files) {
      for (let i = 0; i < _files.length; i++) {
        let reader = new FileReader();
        let vm = this;
        reader.readAsDataURL(_files[i]);

        reader.onload = function(e) {
          vm.files[i] = reader.result;
        };

        if (_files[i].type.match("video/*")) {
          this.generateThumbnail(_files[i]);
        }

        this.error = "";
      }

    },

    removeFile(_index) {
      this.files.splice(_index, 1);
      this.video_thumbnails.splice(_index, 1);
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

.cursor-pointer {
  cursor: pointer;
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

/*Modal Css*/
body {
  font-family: Verdana, sans-serif;
  margin: 0;
}

* {
  box-sizing: border-box;
}

.row > .column {
  padding: 0 8px;
}

.row:after {
  content: "";
  display: table;
  clear: both;
}

.column {
  float: left;
  width: 25%;
}

/* The Modal (background) */
.modal {
  display: none;
  position: fixed;
  z-index: 1;
  padding-top: 100px;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: black;
}

/* Modal Content */
.modal-content {
  position: relative;
  background-color: #fefefe;
  margin: auto;
  padding: 0;
  width: 90%;
  max-width: 1200px;
}

/* The Close Button */
.close {
  color: white;
  position: absolute;
  top: 10px;
  right: 25px;
  font-size: 35px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: #999;
  text-decoration: none;
  cursor: pointer;
}

.cursor {
  cursor: pointer;
}

/* Next & previous buttons */
.prev,
.next {
  cursor: pointer;
  position: absolute;
  top: 50%;
  width: auto;
  padding: 16px;
  margin-top: -50px;
  color: white;
  font-weight: bold;
  font-size: 20px;
  transition: 0.6s ease;
  border-radius: 0 3px 3px 0;
  user-select: none;
  -webkit-user-select: none;
}

/* Position the "next button" to the right */
.next {
  right: 0;
  border-radius: 3px 0 0 3px;
}

/* On hover, add a black background color with a little bit see-through */
.prev:hover,
.next:hover {
  background-color: rgba(0, 0, 0, 0.8);
}

/* Number text (1/3 etc) */
.numbertext {
  color: #f2f2f2;
  font-size: 12px;
  padding: 8px 12px;
  position: absolute;
  top: 0;
}

img {
  margin-bottom: -4px;
}

.caption-container {
  text-align: center;
  background-color: black;
  padding: 2px 16px;
  color: white;
}

.demo {
  opacity: 0.6;
}

.active,
.demo:hover {
  opacity: 1;
}

img.hover-shadow {
  transition: 0.3s;
}

.hover-shadow:hover {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}
</style>

