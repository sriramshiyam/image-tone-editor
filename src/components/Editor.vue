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
      <button :style="styleB('blur')" @click="styleType = 'blur'">blur</button>
      <button :style="styleB('brightness')" @click="styleType = 'brightness'">
        brightness
      </button>
      <button :style="styleB('contrast')" @click="styleType = 'contrast'">
        contrast
      </button>
      <button :style="styleB('grayscale')" @click="styleType = 'grayscale'">
        grayscale
      </button>
      <button :style="styleB('hue-rotate')" @click="styleType = 'hue-rotate'">
        hue-rotate
      </button>
      <button :style="styleB('opacity')" @click="styleType = 'opacity'">
        opacity
      </button>
      <button :style="styleB('saturate')" @click="styleType = 'saturate'">
        saturate
      </button>
      <button :style="styleB('sepia')" @click="styleType = 'sepia'">
        sepia
      </button>
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
    <div v-show="styleType === 'grayscale'">
      <input type="range" min="0" max="100" v-model="grayscale" />
      <h2>{{ grayscale }}</h2>
    </div>
    <div v-show="styleType === 'blur'">
      <input type="range" min="0" max="10" v-model="blur" />
      <h2>{{ blur }}</h2>
    </div>
    <div v-show="styleType === 'brightness'">
      <input type="range" min="0" max="100" v-model="brightness" />
      <h2>{{ brightness }}</h2>
    </div>
    <div v-show="styleType === 'contrast'">
      <input type="range" min="100" max="200" v-model="contrast" />
      <h2>{{ contrast }}</h2>
    </div>
    <div v-show="styleType === 'hue-rotate'">
      <input type="range" min="0" max="360" v-model="hueRotate" />
      <h2>{{ hueRotate }}</h2>
    </div>
    <div v-show="styleType === 'opacity'">
      <input type="range" min="0" max="100" v-model="opacity" />
      <h2>{{ opacity }}</h2>
    </div>
    <div v-show="styleType === 'saturate'">
      <input type="range" min="100" max="300" v-model="saturate" />
      <h2>{{ saturate }}</h2>
    </div>
    <div v-show="styleType === 'sepia'">
      <input type="range" min="0" max="100" v-model="sepia" />
      <h2>{{ sepia }}</h2>
    </div>
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
      blur: 0,
      brightness: 100,
      contrast: 100,
      grayscale: 0,
      hueRotate: 0,
      opacity: 100,
      saturate: 100,
      sepia: 0,
    };
  },
  computed: {
    fil() {
      return `brightness(${this.brightness}%) blur(${this.blur}px) contrast(${this.contrast}%) grayscale(${this.grayscale}%) hue-rotate(${this.hueRotate}deg) opacity(${this.opacity}%) saturate(${this.saturate}%) sepia(${this.sepia}%)`;
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
        this.blur = 0;
        this.brightness = 100;
        this.contrast = 100;
        this.grayscale = 0;
        this.hueRotate = 0;
        this.opacity = 100;
        this.saturate = 100;
        this.sepia = 0;
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
