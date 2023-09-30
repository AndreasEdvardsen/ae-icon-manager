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
          <v-btn
            class="mr-8"
            v-bind="props"
            prepend-icon="mdi-palette"
            :color="color"
            >Color</v-btn
          >
        </template>
        <v-color-picker
          :modes="['hexa']"
          v-model="color"
          show-swatches
        ></v-color-picker>
      </v-menu>
      <v-slider
        :min="10"
        :max="100"
        :step="1"
        thumb-label="always"
        v-model="iconSize"
      ></v-slider>
    </v-row>
    <v-row class="ma-10">
      <v-tooltip v-for="icon in icons" :text="icon.name">
        <template v-slot:activator="{ props }">
          <v-btn
            variant="text"
            size="x-large"
            rounded="xl"
            @click="copyToClipboard(icon)"
            v-bind="props"
          >
            <img
              class="svg"
              v-bind:src="icon.imagePath"
              v-bind:alt="icon.path"
            />
          </v-btn>
        </template>
      </v-tooltip>
    </v-row>
    <v-pagination :length="pageAmmount" v-model="pageNumber"></v-pagination>
  </v-container>
</template>

<script setup>
import { computed, getCurrentInstance } from "vue";
import json from "../assets/library-mdi.json";
import { ref } from "vue";

const searchString = ref("");
const allicons = json;
const color = ref("#FFFFFF");
const iconSize = ref(25);
const pageSize = 100;
const pageNumber = ref(1);

allicons.forEach((i) => {
  var image = encodeURIComponent(
    i.image.replace("<path ", `<path style="fill:${color.value};" `)
  );
  i.imagePath = `data:image/svg+xml;utf8,${image}`;
});

function copyToClipboard(icon) {
  var text = icon.image
    .replace("<path ", `<path style="fill:${color.value};" `) // Add color
    .replace('width="48" height="48" ', "") // Remove sixe from mdi icons
    .replace(
      "<svg",
      `<svg width="${iconSize.value}px" height="${iconSize.value}px"`
    ); // Add size to icons
  navigator.clipboard.writeText(text);
}

const pageAmmount = computed(() => {
  return Math.ceil(allicons.length / pageSize);
});

const icons = computed(() => {
  let i = allicons.filter((icon) => {
    if (icon.path.includes(searchString.value)) return true;
    return false;
  });
  return i.slice(
    (pageNumber.value - 1) * pageSize,
    pageNumber.value * pageSize
  );
});
</script>

<style>
.svg {
  height: 35px;
  width: auto;
}
</style>
