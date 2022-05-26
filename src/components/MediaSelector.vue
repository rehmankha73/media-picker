<template>
  <div :class="[ files.length === 0 ? 'd-flex justify-content-center align-items-center' : '' ]"
       class="drop-box display-inline mt-6"
       @drop="onDrop"
       @dragover.prevent
  >
    <div v-show="files.length === 0" class="label-overview mt-2">
      <div class="">
        <button
          class="file-button"
          type="button"
          @click="files.length === 0 ? $refs['file-input'].click() : ''"
        >
          Add file
        </button>
        <span class="url-link">Add from url</span>
        <p class="label-overview mt-2">
          {{ label }}
        </p>
        <input ref="file-input" class="file-input" multiple name="image" type="file" @change="onChange">
      </div>
    </div>

    <div v-if="files.length > 0 " id="uploads-div" class="preview-div p-2 mt-2 overflow-hidden">
      <div class="row mt-2">
        <div v-for="(file, index) in files" class="col-3 preview-item position-relative">

          <img
            v-if="file && start_with(file.type, 'image') && (this.fileTypes.includes('image/*') || (this.fileTypes.includes(file.type)))"
            :src="file.url"
            alt="logo"
            class="image-preview"
            @click="openModal(file.url, 'image')"
          />

          <img
            v-if="file && start_with(file.type, 'video') && (this.fileTypes.includes('video/*') || (this.fileTypes.includes(file.type)))"
            :src="file.thumbnail.thumbnail"
            alt="logo"
            class="image-preview"
            @click="openModal(file.url, 'video')"
          />

          <embed
            v-else-if="file && start_with(file.type, 'application') && (this.fileTypes.includes('application/*') || (this.fileTypes.includes(file.type)))"
            :src="file.url"
            class="d-block cursor-pointer" style="width: 100%; height: 100%; object-fit:cover;" />

          <button class="mt-2 position-absolute danger-button" style="top: -8px; right: 0px" type="button"
                  @click.self="removeFile(index)">X
          </button>
        </div>

        <div class="col-3 preview-item" style="padding-top: 20px; margin-left: 10px">
          <div class="cursor-pointer">
            <button
              class="file-button"
              type="button"
              @click="$refs['file-input'].click()"
            >
              Add file
            </button>
            <p class="url-link mt-2">Add from url</p>
          </div>
        </div>

      </div>
    </div>

  </div>

  <br>
  <span v-if="error" class="text-danger">{{ error }}</span>

  <div v-if="showAllowedTypes" class="mt-4">
    <span><b>Type of files:</b> {{ allowedTypes }}</span>
  </div>

  Uploaded Files: {{ no_of_files ? no_of_files : 0 }}

  <div id="myModal" class="modal">
    <span class="close cursor-pointer" @click="closeModal()">&times;</span>
    <div class="modal-content">
      <div id="image-slide" style="display: none;background: black">
        <img v-if="start_with" id="modal-image" alt="image" src="../assets/logo.png"
             style="width: 500px; height: 500px;background: black">
      </div>

      <div id="video-slide" style="display: none; background: black">
        <video id="modal-video"
               class="mx-auto block"
               controls style="width: 500px; height: 500px; background: black">
          <source id="modal-video-source" src="" type="video/mp4">
        </video>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "FileSelector",
  props: {
    label: {
      type: String,
      default: "SELECT OR DROP AN IMAGES/FILES"
    },
    showAllowedTypes: {
      type: Boolean,
      default: false
    },
    formErrors: {
      type: String,
      default: "PLease Upload related typed documents which are allowed!"
    },
    allowedTypes: {
      type: String,
      default: "image/*"
    }
  },

  data() {
    return {
      error: "",
      no_of_files: "",
      files: []
    };
  },
  computed: {
    fileTypes() {
      return this.allowedTypes.split(",");
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
  mounted() {
    this.files = [];
  },
  methods: {

    onDrop(e) {
      e.stopPropagation();
      e.preventDefault();
      let _files = e.dataTransfer.files;
      this.uploadFiles(this.validateAndUploadData(_files));
      console.log(this.files, "files");
    },

    onChange(e) {
      let _files = e.target.files;
      this.uploadFiles(this.validateAndUploadData(_files));
      console.log(this.files, "files");
    },

    validateAndUploadData(_files) {
      let _data = [];
      for (let f = 0; f < _files.length; f++) {

        let _file_type = _files[f].type.split("/");
        let _file_starts_with = _file_type[0];

        // type validation for image
        if (!this.fileTypes.includes("image/*")) {
          if (!this.fileTypes.includes(_files[f].type) && _file_starts_with === "image") {
            this.error = "File is not an image, or a selected type of image like image/png";
            alert(this.error);
            break;
          }
        }

        // type validation for Video
        if (!this.fileTypes.includes("video/*")) {
          if (!this.fileTypes.includes(_files[f].type) && _file_starts_with === "video") {
            this.error = "File is not a video, or a selected type of image like video/mp4";
            alert(this.error);
            break;
          }
        }

        // type validation for application
        if (!this.fileTypes.includes("application/*")) {
          if (!this.fileTypes.includes(_files[f].type) && _file_starts_with === "application") {
            this.error = "File is not a document, or a selected type of image like application/pdf";
            alert(this.error);
            break;
          }
        }

        _data.push(_files[f]);
      }
      return _data;
    },


    async uploadFiles(files) {
      this.no_of_files = 0;
      await this.handleMedia(files);
      this.files = [...this.files];
    },

    async handleMedia(files) {
      for (let i = 0; i < files.length; i++) {
        // if ((files[i].type.includes("image") && files[i].size < 5242880) || (files[i].type.includes("video") && files[i].size < 209715200)) {
        try {
          const fileData = {
            file: files[i],
            url: URL.createObjectURL(files[i]),
            name: files[i].name,
            type: files[i].type,
            size: files[i].size
          };

          if (fileData.type.includes("video")) {
            const thumbnail = {
              file: null,
              thumbnail: null
            };

            thumbnail.file = await this.generateThumbnail(files[i]);
            thumbnail["thumbnail"] = URL.createObjectURL(thumbnail.file);
            fileData.thumbnail = thumbnail;
          }

          this.files.push(fileData);
          this.no_of_files = this.files.length;

          this.error = "";
        } catch (e) {
          console.log("Some error occurred. " + files[i].name + " cannot be selected.");
        }
        // } else {
        //   console.log(files[i].type.includes("image") ? files[i].name + " is too large. Max size allowed is 5 Mb" : files[i].name + " is too large. Max size allowed is 200 Mb");
        // }
      }
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
              resolve(blob);
            });
          });
          console.log(blob);
          resolve(blob);
        };
      });

      console.log(thumbnail, "thumb");
      return thumbnail;
    },

    removeFile(_index) {
      this.files.splice(_index, 1);
    },

    openModal(_file, _type) {
      let image_slide = document.getElementById("image-slide");
      let modal_image = document.getElementById("modal-image");
      let video_slide = document.getElementById("video-slide");
      let modal_video = document.getElementById("modal-video");
      let modal_video_source = document.getElementById("modal-video-source");

      if (_type && _type === "image") {
        image_slide.style.display = "block";
        video_slide.style.display = "none";

        modal_image.src = _file;
      }

      if (_type && _type === "video") {
        video_slide.style.display = "block";
        image_slide.style.display = "none";

        modal_video_source.setAttribute("src", _file);

        modal_video.load();
      }
      document.getElementById("myModal").style.display = "block";
    },

    closeModal() {
      document.getElementById("myModal").style.display = "none";
    },

    start_with(_type, _start_with) {
      let file_start_with = _type.split("/");
      return file_start_with[0] === _start_with;
    },

  }
};
</script>

<style>
* {
  font-family: 'Arial', serif;
  font-size: 12px;
  box-sizing: border-box;
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
  font-family: Verdana, sans-serif;
  margin: 0;
}

.preview-div {
  margin-right: 10px;
  margin-left: 10px;
}

.preview-item {
  border: 1px solid lightgrey;
  padding: 5px;
  height: 85px;
  border-radius: 5%;
}

.file-button {
  background: lightblue;
  color: blue;
  padding: 5px 10px;
  border-radius: 5%;
  border: none;
  outline: none;
}

.danger-button {
  background: lightcoral;
  color: red;
  border-radius: 5%;
  border: none;
  outline: none;
}

.row {
  margin: 0px;
}

.danger-button:hover {
  color: white;
  background: red;
  border: none;
  outline: none;
}

.file-button:hover {
  color: white;
  background: blue;
  padding: 5px 10px;
  border: none;
  outline: none;
}

.label-overview {
  width: 100%;
  font-size: 12px;
  color: lightgrey;
}

.url-link {
  margin-left: 5px;
  color: blue;
  text-decoration: underline;
}

input[type="file"] {
  position: absolute;
  opacity: 0;
  z-index: -1;
}

.align-center {
  text-align: center;
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
  max-height: 100%;
  max-width: 100%;
  width: auto;
  object-fit: cover;
}

.drop-box {
  text-align: center;
  background-color: #f2f2f2;
  border: 2px dashed #ccc;
  border-radius: 2px;
  width: 100%;
  height: 300px;
  cursor: pointer;
}

.file-input {
  display: none;
}


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

.modal-content {
  position: relative;
  background-color: #fefefe;
  margin: auto;
  padding: 0;
  width: 90%;
  max-width: 1200px;
}

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

img {
  margin-bottom: -4px;
}

img.hover-shadow {
  transition: 0.3s;
}

.hover-shadow:hover {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

</style>

