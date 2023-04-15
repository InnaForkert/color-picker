<script setup>
import { reactive, computed, ref } from "vue";

const btnText = ref('Copy CSS');
const colorPicker = ref(null);

const color = reactive({
  fromRange: 0,
  hex: "#ffffff",
  r: 255,
  g: 255,
  b: 255,
  a: 1,
});

const finalColor = reactive({
  ...color,
});

const coords = reactive({
  x: 0,
  y: 0,
});

const x = computed(() => `${coords.x - 12.5}px`);
const y = computed(() => `${coords.y - 12.5}px`);

const bcgColor = computed(() => `linear-gradient(${gradient.angle}deg, ${gradient.firstColor} ${gradient.stop}%, ${gradient.secondColor}, ${gradient.thirdColor})`)

const chosenColor = computed(
  () => `rgba(${color.r}, ${color.g}, ${color.b}, ${finalColor.a})`
);

const chosenColor100 = computed(
  () => `rgb(${color.r}, ${color.g}, ${color.b})`
);
const chosenColor80 = computed(
  () => `rgba(${color.r}, ${color.g}, ${color.b}, 0.8)`
);
const chosenColor60 = computed(
  () => `rgba(${color.r}, ${color.g}, ${color.b}, 0.6)`
);

const gradient = reactive({
  angle: "39",
  firstColor: "#245db6",
  secondColor: "#9b2098",
  thirdColor: "#f2c21c",
  activeColor: "secondColor",
  stop: 30,
});

const angle = computed(() => `${gradient.angle}deg`);
const firstColor = computed(() => gradient.firstColor);
const secondColor = computed(() => gradient.secondColor);
const thirdColor = computed(() => gradient.thirdColor);
const stop = computed(() => `${gradient.stop}%`);

let computedFinalColor = chosenColor;

function dragPicker({ offsetX, offsetY, buttons }) {
  if (buttons === 1) {
    coords.x = offsetX;
    coords.y = offsetY;
    if (coords.x < 0) coords.x = 0;
    if (coords.y < 0) coords.y = 0;
    if (coords.x > colorPicker.value.clientWidth) coords.x = colorPicker.value.clientWidth;
    if (coords.y > colorPicker.value.clientHeight) coords.y = colorPicker.value.clientHeight;
    computeFinalColor((offsetX / colorPicker.value.clientWidth), (offsetY / colorPicker.value.clientHeight));
    computeGradient();
    computeHex();
  }
}

function computeColorFromRange() {
  if (color.fromRange < 20) {
    color.r = 255;
    color.g = Math.round(color.fromRange * 12.75);
    color.b = 0;
  } else if (color.fromRange < 40) {
    color.r = Math.round(255 - (40 - color.fromRange) * 12.75);
    color.g = 255;
    color.b = 0;
  } else if (color.fromRange < 60) {
    color.r = 0;
    color.g = 255;
    color.b = Math.round((60 - color.fromRange) * 12.75);
  } else if (color.fromRange < 80) {
    color.r = 0;
    color.g = Math.round(255 - (80 - color.fromRange) * 12.75);
    color.b = 255;
  } else if (color.fromRange < 100) {
    color.r = Math.round((100 - color.fromRange) * 12.75);
    color.g = 0;
    color.b = 255;
  } else {
    color.r = 255;
    color.g = 0;
    color.b = Math.round(255 - (120 - color.fromRange) * 12.75);
  }
  computeHex();
  computeGradient();
  computeFinalColor(coords.x, coords.y);
}

function copyGradient() {
  navigator.clipboard.writeText(bcgColor.value)
    .then(() => {
      btnText.value = 'Copied!';
      setTimeout(() => btnText.value = 'Copy CSS', 2500);
    })
    .catch((err) => {
      console.error('Unable to copy to clipboard:', err);
    });
}

function computeHex() {
  finalColor.hex = `#${finalColor.r.toString(16).padStart(2, '0')}${finalColor.g.toString(
    16
  ).padStart(2, '0')}${finalColor.b.toString(16).padStart(2, '0')}`;
}

function handleHexBlur() {
  finalColor.r = parseInt(finalColor.hex.slice(1, 3), 16);
  finalColor.g = parseInt(finalColor.hex.slice(3, 5), 16);
  finalColor.b = parseInt(finalColor.hex.slice(5), 16);
  computeGradient();
}

function computeGradient() {
  gradient[
    gradient.activeColor
  ] = `rgba(${finalColor.r}, ${finalColor.g}, ${finalColor.b}, ${finalColor.a})`;
}

function choseActiveColor(i) {
  gradient.activeColor = i;
  computeGradient();
}

function computeFinalColor(x, y) {
  const computedColorX = {
    r: color.r * x,
    g: color.g * x,
    b: color.b * x,
  };

  finalColor.r = Math.round(computedColorX.r - (computedColorX.r * y) / 90);
  finalColor.g = Math.round(computedColorX.g - (computedColorX.g * y) / 90);
  finalColor.b = Math.round(computedColorX.b - (computedColorX.b * y) / 90);

  computedFinalColor = computed(
    () =>
      `rgba(${finalColor.r}, ${finalColor.g}, ${finalColor.b}, ${finalColor.a})`
  );

  computeHex();
}
</script>


<script>
export default {
  name: 'ColorPicker',
}
</script>
<template>
  <div class="color-picker-container">
    <div class="color-picker-section">
      <div class="color-picker" @pointermove="dragPicker" ref="colorPicker"></div>
      <div class="inputs-preview">
        <div class="inputs">
          <input type="range" class="color-range" v-model="color.fromRange" min="0" max="120" step="1"
            @input="computeColorFromRange" />
          <input type="range" class="opacity-range" v-model="finalColor.a" min="0" max="1" step="0.1"
            @input="computeGradient" />
        </div>
        <div class="color-preview"></div>
      </div>
      <div class="color-values">
        <label>Hex
          <input type="text" v-model="finalColor.hex" @blur="handleHexBlur" /></label>
        <label>R
          <input type="number" v-model="finalColor.r" @input="computeGradient" /></label>
        <label>G
          <input type="number" v-model="finalColor.g" @input="computeGradient" /></label>
        <label>B
          <input type="number" v-model="finalColor.b" @input="computeGradient" /></label>
        <label>A
          <input type="number" v-model="finalColor.a" @input="computeGradient" /></label>
      </div>
    </div>
    <div class="color-picker-section">
      <div class="gradient-preview"></div>
      <label class="gradient-range-label">
        <div class="before" @click="choseActiveColor('firstColor')"></div>
        <input type="range" class="gradient-range" @input="choseActiveColor('secondColor')" v-model="gradient.stop"
          min="0" max="100" step="1" />
        <div class="after" @click="choseActiveColor('thirdColor')"></div>
      </label>
      <div class="angle-inputs">
        <label>Angle<input type="range" class="angle-range" v-model="gradient.angle" @input="computeGradient" min="0"
            max="360" step="1" /></label>
        <label>Deg°<input type="number" v-model="gradient.angle" @input="computeGradient" min="0" max="360"
            step="1" /></label>
      </div>
      <button class="btn" @click="copyGradient">{{ btnText }}</button>
    </div>
  </div>
</template>

<style lang="css">
body {
  overflow: hidden;
}

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

.btn {
  position: relative;
  width: 100px;
  top: 50px;
  right: 50px;
  align-items: center;
  appearance: none;
  background-color: #fff;
  border-radius: 24px;
  border-style: none;
  box-shadow: rgba(0, 0, 0, .2) 0 3px 5px -1px, rgba(0, 0, 0, .14) 0 6px 10px 0, rgba(0, 0, 0, .12) 0 1px 18px 0;
  box-sizing: border-box;
  color: #3c4043;
  cursor: pointer;
  display: inline-flex;
  fill: currentcolor;
  font-family: "Google Sans", Roboto, Arial, sans-serif;
  font-size: 14px;
  font-weight: 500;
  height: 48px;
  justify-content: center;
  letter-spacing: .25px;
  line-height: normal;
  overflow: visible;
  position: relative;
  text-align: center;
  text-transform: none;
  transition: box-shadow 280ms cubic-bezier(.4, 0, .2, 1), opacity 15ms linear 30ms, transform 270ms cubic-bezier(0, 0, .2, 1) 0ms;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  will-change: transform, opacity;
  z-index: 0;
}

.btn:hover {
  background: #F6F9FE;
  color: #174ea6;
}

.btn:active {
  box-shadow: 0 4px 4px 0 rgb(60 64 67 / 30%), 0 8px 12px 6px rgb(60 64 67 / 15%);
  outline: none;
}

.btn:focus {
  outline: none;
  border: 2px solid #4285f4;
}

.btn:not(:disabled) {
  box-shadow: rgba(60, 64, 67, .3) 0 1px 3px 0, rgba(60, 64, 67, .15) 0 4px 8px 3px;
}

.btn:not(:disabled):hover {
  box-shadow: rgba(60, 64, 67, .3) 0 2px 3px 0, rgba(60, 64, 67, .15) 0 6px 10px 4px;
}

.btn:not(:disabled):focus {
  box-shadow: rgba(60, 64, 67, .3) 0 1px 3px 0, rgba(60, 64, 67, .15) 0 4px 8px 3px;
}

.btn:not(:disabled):active {
  box-shadow: rgba(60, 64, 67, .3) 0 4px 4px 0, rgba(60, 64, 67, .15) 0 8px 12px 6px;
}

.btn:disabled {
  box-shadow: rgba(60, 64, 67, .3) 0 1px 3px 0, rgba(60, 64, 67, .15) 0 4px 8px 3px;
}

.color-picker-container {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
  background: v-bind(bcgColor);
}

.color-picker-section {
  width: 40vw;
  height: 30vh;
}

input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);

}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  border: 1px solid #00000042;
  height: 100%;
  width: 15px;
  border-radius: 3px;
  background: #ffffff;
  cursor: pointer;
  z-index: 1;
  position: relative;
}

input[type="range"]::-ms-track {
  width: 100%;
  cursor: pointer;
  background: transparent;
  border-color: transparent;
  color: transparent;
  height: 100%;
}

input[type="range"]::-webkit-slider-runnable-track {
  cursor: pointer;
  width: 100%;
  appearance: none;
  height: 20px;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}


.color-picker {
  position: relative;

  width: 100%;
  height: 100%;
  margin-bottom: 4px;
  border: 2px solid white;
  border-radius: 10px;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);

  cursor: pointer;

  background: radial-gradient(circle at 0 0,
      white 1%,
      #ffffffa2 20%,
      #ffffff57 40%,
      rgba(0, 0, 0, 0) 70%),
    radial-gradient(circle at 100% 0%,
      v-bind(chosenColor100) 1%,
      v-bind(chosenColor80) 20%,
      v-bind(chosenColor60) 40%,
      rgba(0, 0, 0, 0) 80%),
    radial-gradient(circle at 100% 100%, rgb(0, 0, 0), #b5049200 70%),
    radial-gradient(circle at 0% 100%,
      rgb(0, 0, 0) 1%,
      rgba(0, 0, 0, 0.849) 20%,
      rgba(0, 0, 0, 0.295) 40%,
      #00000000 100%),
    linear-gradient(45deg, black, v-bind(chosenColor100)),
    linear-gradient(to top, black 40%, white 60%);
}

.color-picker::after {
  content: "";

  position: absolute;
  top: v-bind(y);
  left: v-bind(x);

  display: block;

  width: 25px;
  height: 25px;

  border: 2px solid white;
  border-radius: 50%;

  background-color: transparent;

  cursor: pointer;
}

.inputs-preview {
  display: flex;
  gap: 12px;
  width: 100%;
  justify-content: center;
}

.inputs {
  display: flex;
  flex-direction: column;
  gap:
    4px;
  width: 100%;
}

.color-range {
  border-radius: 4px;
  overflow: hidden;
  border: 1px solid white;
}

.color-range::-webkit-slider-runnable-track {
  background: linear-gradient(90deg,
      #ff0000,
      #ff6a00,
      #ffc800,
      #9dff00,
      #0dff00,
      #00ff80,
      #00ffff,
      #006aff,
      #1500ff,
      #bb00ff,
      #ff00c8,
      #ff003c,
      #ff0000);
}

.opacity-range {
  position: relative;

  background-image: url("https://t3.ftcdn.net/jpg/03/76/74/78/360_F_376747823_L8il80K6c2CM1lnPYJhhJZQNl6ynX1yj.jpg");
  background-position-y: 46%;
  background-size: 10%;

  overflow: hidden;

  border: 1px solid white;
  border-radius: 4px;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);
}

.opacity-range::after {
  content: "";

  position: absolute;
  top: 0;
  left: 0;

  display: block;

  width: 100%;
  height: 100%;

  background: linear-gradient(90deg, transparent, v-bind(chosenColor100));
}

.color-preview {
  width: 50px;
  height: 50px;

  border-radius: 4px;
  border: 1px solid white;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);

  background-color: v-bind(computedFinalColor);
}

.color-values {
  display: grid;
  grid-template-columns: 2.5fr 1fr 1fr 1fr 1fr;
  gap: 25px;

  font-size: 18px;
  font-family: Monospace;
  text-align: center;
}

.color-values label {
  display: flex;
  flex-direction: column-reverse;

}

.color-values label input {
  width: 100%;
  height: 24px;
  padding: 5px;
  outline: none;
  border-radius: 10px;
  outline: none;
  border: none;
  box-shadow: inset 1px 1px 1px black;
  margin-top: 4px;
  font-size: 18px;
  font-family: Monospace;
  text-align: center;
}

.gradient-preview {
  width: 100%;
  height: 100%;
  margin-bottom: 4px;
  border: 2px solid white;
  border-radius: 10px;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);

  background: linear-gradient(v-bind(angle),
      v-bind(firstColor) v-bind(stop),
      v-bind(secondColor),
      v-bind(thirdColor));
}

.gradient-range-label {
  position: relative;
}

.before {
  content: "";
  position: absolute;
  bottom: 0px;
  left: -2px;

  display: block;

  width: 15px;
  height: 15px;

  border: 2px solid white;
  outline: 1px solid rgba(0, 0, 0, 0.152);
  border-radius: 50%;

  background-color: v-bind(firstColor);

  cursor: pointer;
}

.after {
  content: "";
  position: absolute;
  bottom: 0px;
  right: -2px;

  display: block;

  width: 15px;
  height: 15px;

  border: 2px solid white;
  outline: 1px solid rgba(0, 0, 0, 0.152);
  border-radius: 50%;

  background-color: v-bind(thirdColor);

  cursor: pointer;
}

input.gradient-range {
  border-radius: 4px;
  border: 1px solid white;
  background: linear-gradient(90deg,
      v-bind(firstColor) v-bind(stop),
      v-bind(secondColor),
      v-bind(thirdColor));
}

input.gradient-range::-webkit-slider-runnable-track {
  border-radius: 2px;
  height: 100%;
}

input.gradient-range::-webkit-slider-thumb {
  position: relative;
  z-index: 1;

  height: 15px;
  width: 15px;
  margin-top: 5px;

  border: 2px solid white;
  outline: 1px solid rgba(0, 0, 0, 0.184);
  border-radius: 50%;

  background: v-bind(firstColor);

  cursor: pointer;

}

.angle-inputs {
  display: grid;
  grid-template-columns: 80% 20%;
  gap: 5px;
  align-items: center;

  width: 100%;

  text-align: center;

  outline: none;

  font-size: 18px;
  font-family: Monospace;
  text-align: center;
}

.angle-inputs label {
  display: flex;
  flex-direction: column-reverse;
  gap: 2px;
  align-items: center;
}

.angle-inputs label input[type='number'] {
  width: 100%;
  height: 24px;
  padding: 5px;
  outline: none;
  border-radius: 10px;
  outline: none;
  border: none;
  box-shadow: inset 1px 1px 1px black;
  font-size: 18px;
  font-family: Monospace;
  text-align: center;
}

.angle-range {
  height: 100%;
  width: 100%;
  border: 1px solid white;
  border-radius: 4px;
  box-shadow: 2px 2px 2px rgba(0, 0, 0, 0.747);
}

.angle-range::-webkit-slider-runnable-track {
  border: 1px solid rgba(128, 128, 128, 0.438);
  background-color: #f1f1f1;
  height: 100%;
}
</style>
