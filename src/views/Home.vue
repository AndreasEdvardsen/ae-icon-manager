<template>
  <v-container fluid>
    <v-row>
      <v-col>
        <v-text-field
          label="Search For Icon"
          prepend-inner-icon="mdi-magnify"
          single-line
          variant="solo"
          density="compact"
          v-model="searchString"
        ></v-text-field>
      </v-col>
    </v-row>
    <v-row class="pl-3 pr-1">
      <v-menu>
        <template v-slot:activator="{ props }">
          <v-btn v-bind="props" prepend-icon="mdi-palette" :color="color"
            >Color</v-btn
          >
        </template>
        <v-color-picker
          :modes="['hexa']"
          v-model="color"
          show-swatches
        ></v-color-picker>
      </v-menu>
      <v-radio-group
        inline
        v-model="selectedPrefix"
        class="flex-shrink-1 ml-md-2 ml-xs-0 mr-4"
        style="flex: unset"
      >
        <v-radio
          v-for="prefix in prefixes"
          :label="prefix"
          :value="prefix"
        ></v-radio>
      </v-radio-group>
      <v-slider
        min-width="100px"
        :min="10"
        :max="100"
        :step="1"
        thumb-label="always"
        v-model="iconSize"
        class="flex-grow-1"
      ></v-slider>
    </v-row>
    <v-row justify="center">
      <v-col v-if="!loading" v-for="icon in icons" cols="auto">
        <v-tooltip :text="icon.name">
          <template v-slot:activator="{ props }">
            <v-btn
              variant="text"
              icon
              size="large"
              rounded="xl"
              min-width="55px"
              @click="copyToClipboard(icon)"
              v-bind="props"
            >
              <span
                class="svg"
                v-html="icon.image"
                v-bind:alt="icon.path"
              ></span>
            </v-btn>
          </template>
        </v-tooltip>
      </v-col>
      <v-col v-if="loading && !serverDown" class="d-flex justify-center">
        <v-progress-circular indeterminate></v-progress-circular>
      </v-col>
      <v-col v-if="serverDown">
        <v-alert icon="mdi-alert"> Server is down! </v-alert>
      </v-col>
    </v-row>
    <v-pagination
      class="mt-8"
      :length="pageAmmount"
      v-model="pageNumber"
    ></v-pagination>
    <v-snackbar
      min-width="0px"
      color="black"
      rounded="pill"
      :timeout="1000"
      v-model="snackbar"
    >
      <v-icon icon="mdi-emoticon-cool"></v-icon>
      Icon copied to clipboard!
    </v-snackbar>
  </v-container>
</template>

<script setup>
import { computed, getCurrentInstance, reactive, watch } from "vue";
import { ref } from "vue";
import axios from "axios";

const searchString = ref("");
var allicons = ref([]);
const color = ref("#FFFFFF");
const iconSize = ref(25);
const pageSize = 100;
const pageNumber = ref(1);
const pageAmmount = ref(0);
const snackbar = ref(false);
const prefixes = ref(["mdi"]);
const selectedPrefix = ref("mdi");
const loading = ref(true);
const serverDown = ref(false);

getDistinctIconPrefixes();
getIcons(selectedPrefix);

watch(selectedPrefix, () => {
  getIcons(selectedPrefix);
});

const icons = computed(() => {
  if (allicons.value.length == 0) return [];
  var filteredIcons = filterIcons(allicons.value);
  pageAmmount.value = Math.ceil(filteredIcons.length / pageSize);

  if (pageNumber.value > pageAmmount.value) {
    pageNumber.value = 1;
  }

  var iconsOnPage = filteredIcons.slice(
    (pageNumber.value - 1) * pageSize,
    pageNumber.value * pageSize
  );
  //Set icon color
  iconsOnPage.forEach((icon) => {
    if (icon.isOutline) {
      icon.image = updateSVGAttribute(icon.image, "stroke", `${color.value}`);
    } else {
      icon.image = updateSVGAttribute(icon.image, "fill", `${color.value}`);
    }
  });
  return iconsOnPage;
});

function filterIcons(icons) {
  var filteredIcons = icons.filter((icon) => {
    if (icon.path.includes(searchString.value)) return true;
    return false;
  });
  return filteredIcons;
}

function getDistinctIconPrefixes() {
  axios
    .get("http://localhost:4000/distinctIconPrefixes")
    .then(function (response) {
      prefixes.value = response.data;
    });
}

function getIcons(prefix) {
  loading.value = true;
  axios
    .get(`http://localhost:4000/icons?prefix=${prefix.value}`)
    .then(function (response) {
      allicons.value = response.data;
      loading.value = false;
    })
    .catch(function (error) {
      console.log(error);
      loading.value = false;
      serverDown.value = true;
    });
}

function copyToClipboard(icon) {
  var text = icon.image;
  if (icon.isOutline) {
    text = updateSVGAttribute(text, "stroke", `${color.value}`);
  } else {
    text = updateSVGAttribute(text, "fill", `${color.value}`);
  }
  text = updateSVGAttribute(text, "width", `${iconSize.value}`);
  text = updateSVGAttribute(text, "height", `${iconSize.value}`);
  navigator.clipboard.writeText(text);
  snackbar.value = true;
}

function updateSVGAttribute(svgString, svgAttribute, value) {
  // Parse the SVG string into a DOM object
  const parser = new DOMParser();
  const serializer = new XMLSerializer();
  const doc = parser.parseFromString(svgString, "image/svg+xml");
  const svgElement = doc.querySelector("svg");
  const svgElementChildren = doc.querySelectorAll(`[${svgAttribute}]`);

  // If Children with the attribute already exist, update them, otherwise add the attribute to the parent
  if (svgElementChildren.length > 0) {
    svgElementChildren.forEach((element) => {
      element.setAttribute(svgAttribute, value);
    });
  } else {
    svgElement.setAttribute(svgAttribute, value);
  }

  // Serialize the updated DOM object back into a string
  const updatedSVGString = serializer.serializeToString(svgElement);

  return updatedSVGString;
}
</script>

<style>
.svg svg {
  height: 40px;
  width: auto;
}
</style>
