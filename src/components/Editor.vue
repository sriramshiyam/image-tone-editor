<template>
  <div v-show="state === 'upload'" id="upload">
    <h1>Image Tone Editor</h1>
    <button @click="stateChange">
      Upload Image
      <svg
        stroke="currentColor"
        fill="currentColor"
        stroke-width="0"
        viewBox="0 0 24 24"
        height="1em"
        width="1em"
        xmlns="http://www.w3.org/2000/svg"
      >
        <circle cx="7.499" cy="9.5" r="1.5"></circle>
        <path
          d="M10.499 14L8.999 12 5.999 16 8.999 16 11.999 16 17.999 16 13.499 10z"
        ></path>
        <path
          d="M19.999,4h-16c-1.103,0-2,0.897-2,2v12c0,1.103,0.897,2,2,2h16c1.103,0,2-0.897,2-2V6C21.999,4.897,21.102,4,19.999,4z M3.999,18V6h16l0.002,12H3.999z"
        ></path>
      </svg>
    </button>
  </div>
  <div id="editor" v-show="state === 'editor'">
    <div id="styleType">
      <template v-for="type in types" :key="type">
        <button :style="styleB(type)" @click="styleType = type">
          {{ type }}
        </button>
      </template>
    </div>
    <input
      @change="inp"
      ref="input"
      type="file"
      name=""
      id="image_input"
      accept="image/*"
    />
    <img :style="{ filter: fil }" id="display_image" ref="display_image" />
    <template v-for="type in types" :key="type">
      <div v-show="styleType === type">
        <input
          type="range"
          :min="units[type][0]"
          :max="units[type][1]"
          v-model="units[type][2]"
        />
        <h2>{{ units[type][2] }}</h2>
      </div>
    </template>
    <button id="download" @click="download">Download</button>
  </div>
  <img
    src=""
    :style="{ filter: fil }"
    ref="downloadImage"
    id="downloadImage"
    alt=""
  />
</template>

<script>
import domtoimage from "dom-to-image";

export default {
  name: "Editor",
  data() {
    return {
      imageName: "",
      imageType: "",
      state: "upload",
      styleType: "",
      types: [
        "grayscale",
        "blur",
        "brightness",
        "contrast",
        "hueRotate",
        "opacity",
        "saturate",
        "sepia",
      ],
      units: {
        grayscale: [0, 100, 0],
        blur: [0, 100, 0],
        brightness: [0, 100, 100],
        contrast: [100, 200, 100],
        hueRotate: [0, 360, 0],
        opacity: [0, 100, 100],
        saturate: [100, 300, 100],
        sepia: [0, 100, 0],
      },
    };
  },
  computed: {
    fil() {
      return `brightness(${this.units.brightness[2]}%) blur(${this.units.blur[2]}px) contrast(${this.units.contrast[2]}%) grayscale(${this.units.grayscale[2]}%) hue-rotate(${this.units.hueRotate[2]}deg) opacity(${this.units.opacity[2]}%) saturate(${this.units.saturate[2]}%) sepia(${this.units.sepia[2]}%)`;
    },
  },
  methods: {
    styleB(name) {
      let st = this.styleType;
      return {
        background: st === name ? "white" : "#ff6363",
        color: st === name ? "#ff6363" : "white",
      };
    },
    stateChange() {
      this.$refs.input.click();
      let i = setInterval(() => {
        if (this.$refs.input.files[0]) {
          this.state = "editor";
          clearInterval(i);
        } else {
          console.log("upload");
          this.state = "upload";
        }
      }, 200);
      if (this.$refs.input.files[0]) {
        this.state = "editor";
      } else {
        this.state = "upload";
      }
    },
    inp($event) {
      const reader = new FileReader();
      reader.addEventListener("load", () => {
        let ui = reader.result;
        this.$refs.display_image.src = ui;
        this.$refs.downloadImage.src = ui;
        var image = new Image();
        let height;
        let width;
        //Set the Base64 string return from FileReader as source.
        image.src = reader.result;
        // console.log("height:", image.height, "width:", image.width);

        //Validate the File Height and Width.
        image.onload = function () {
          height = this.height;
          width = this.width;
          console.log("height:", height, "width:", width);
          return true;
        };
        //
        this.$refs.downloadImage.style.height = height + "px";
        this.$refs.downloadImage.style.width = width + "px";
      });
      //
      reader.readAsDataURL($event.target.files[0]);
      this.imageName = $event.target.files[0].name;
      this.imageType = $event.target.files[0].type.split("/")[1];
      console.log(this.imageName, ",type:", this.imageType);
      //

      setTimeout(() => {
        reader.removeEventListener("load", () => {
          let ui = reader.result;
          this.$refs.display_image.src = ui;
          this.$refs.downloadIMage.src = ui;
        });
      }, 1000);
      //
    },
    download() {
      let name = this.imageName;
      if (this.imageType === "jpeg") {
        domtoimage
          .toJpeg(document.getElementById("downloadImage"))
          .then((dataUrl) => {
            var link = document.createElement("a");
            link.download = name;
            link.href = dataUrl;
            link.click();
          });
      } else if (this.imageType === "png") {
        domtoimage
          .toPng(document.getElementById("downloadImage"))
          .then((dataUrl) => {
            var link = document.createElement("a");
            link.download = name;
            link.href = dataUrl;
            link.click();
          });
      }
      setTimeout(() => {
        this.state = "upload";
        this.styleType = "";
        this.units.blur[2] = 0;
        this.units.brightness[2] = 100;
        this.units.contrast[2] = 100;
        this.units.grayscale[2] = 0;
        this.units.hueRotate[2] = 0;
        this.units.opacity[2] = 100;
        this.units.saturate[2] = 100;
        this.units.sepia[2] = 0;
      }, 1000);
    },
  },
};
</script>

<style>
body {
  overflow: hidden;
  margin: 0;
}

#app {
  overflow: hidden;
  font-family: "Anton", sans-serif;
  height: 100vh;
}

#editor,
#upload {
  text-align: center;
  width: 100vw;
  min-height: 100vh;
}

#upload {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-size: 3rem;
}

#upload h1 {
  font-size: 4rem;
  letter-spacing: 2px;
}

#upload button {
  font-family: "Anton", sans-serif;
  padding: 1em;
  font-size: 0.5em;
  border: 0;
  background: #ff6363;
  color: white;
  border-radius: 10px;
  cursor: pointer;
  letter-spacing: 1px;
}

#download {
  font-family: "Anton", sans-serif;
  padding: 1em;
  font-size: 1rem;
  border: 0;
  background: #06ff00;
  border-radius: 10px;
  color: white;
  cursor: pointer;
  letter-spacing: 1px;
}

#editor input[type="file"] {
  display: none;
}

#editor {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow: scroll;
}

#styleType {
  display: flex;
  justify-content: center;
  align-items: center;
  align-content: center;
  flex-wrap: wrap;
}

#styleType button {
  font-family: "Anton", sans-serif;
  padding: 1em;
  font-size: 1rem;
  border: 0;
  cursor: pointer;
  letter-spacing: 1px;
}

input[type="range"] {
  -webkit-appearance: none;
  width: 500px;
  max-width: 90vw;
}
input[type="range"]:focus {
  outline: none;
}
input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 13px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #000000;
  background: #ff6363;
  border-radius: 0px;
  border: 0px solid #000000;
}
input[type="range"]::-webkit-slider-thumb {
  box-shadow: 0px 0px 1px #000000;
  border: 1px solid #000000;
  height: 18px;
  width: 18px;
  border-radius: 1px;
  background: #ffffff;
  cursor: pointer;
  -webkit-appearance: none;
  margin-top: -3px;
}
input[type="range"]:focus::-webkit-slider-runnable-track {
  background: #ff6363;
}
input[type="range"]::-moz-range-track {
  width: 100%;
  height: 13px;
  cursor: pointer;
  box-shadow: 0px 0px 0px #000000;
  background: #ff6363;
  border-radius: 0px;
  border: 0px solid #000000;
}
input[type="range"]::-moz-range-thumb {
  box-shadow: 0px 0px 1px #000000;
  border: 1px solid #000000;
  height: 18px;
  width: 18px;
  border-radius: 1px;
  background: #ffffff;
  cursor: pointer;
}
input[type="range"]::-ms-track {
  width: 100%;
  height: 13px;
  cursor: pointer;
  background: transparent;
  border-color: transparent;
  color: transparent;
}
input[type="range"]::-ms-fill-lower {
  background: #ff6363;
  border: 0px solid #000000;
  border-radius: 0px;
  box-shadow: 0px 0px 0px #000000;
}
input[type="range"]::-ms-fill-upper {
  background: #ff6363;
  border: 0px solid #000000;
  border-radius: 0px;
  box-shadow: 0px 0px 0px #000000;
}
input[type="range"]::-ms-thumb {
  box-shadow: 0px 0px 1px #000000;
  border: 1px solid #000000;
  height: 18px;
  width: 18px;
  border-radius: 1px;
  background: #ffffff;
  cursor: pointer;
}
input[type="range"]:focus::-ms-fill-lower {
  background: #ff6363;
}
input[type="range"]:focus::-ms-fill-upper {
  background: #ff6363;
}

#display_image {
  height: 300px;
  width: 400px;
  max-width: 90vw;
  background-position: center;
  background-size: cover;
  object-fit: contain;
}

#downloadImage {
  z-index: -5;
}

@media screen and (max-width: 720px) {
  #upload h1 {
    font-size: 2rem;
  }
  #styleType button {
    padding: 0.8em;
    font-size: 0.8rem;
  }
}
</style>
