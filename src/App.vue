<script setup>
import { onMounted, ref, watch } from "vue";

const title = ref("");
const image = ref("preview");
const type = ref("");
const url = ref("");
const author = ref("");
const description = ref("");
const baseURL = ref("");
const host = ref("");
const isImageLoaded = ref(false);
const imageLoaded = ref(true);

const handleImageLoad = () => {
  isImageLoaded.value = true;
};

watch(isImageLoaded, () => {
  imageLoaded.value = isImageLoaded.value;
});

const details = async () => {
  const [tab] = await chrome.tabs.query({ active: true });
  chrome.scripting.executeScript({
    target: { tabId: tab.id },
    function: () => {
      const pageTitle = document.querySelector("meta[property='og:title']");
      const pageType = document.querySelector("meta[property='og:type']");
      const pageImage = document.querySelector('meta[property="og:image"]');
      const pageDescription = document.querySelector(
        'meta[property="og:description"]'
      );
      const pageAuthor = document.querySelector('meta[name="author"]');

      const url = document.URL;
      const baseURL = window.location.origin;
      const host = new URL(url).host;

      // const metas = document.head.getElementsByTagName("meta");

      chrome.runtime.sendMessage({
        type: "UPDATE_PAGE_INFO",
        info: {
          title: pageTitle ? pageTitle.getAttribute("content") : "No title",
          image: pageImage ? pageImage.getAttribute("content") : "No image",
          author: pageAuthor ? pageAuthor.getAttribute("content") : "No author",
          url,
          baseURL,
          host,
          type: pageType ? pageType.getAttribute("content") : "No type",
          description: pageDescription
            ? pageDescription.getAttribute("content")
            : "No description",
        },
      });
    },
  });
};

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  console.log(message);
  title.value = message.info.title;
  image.value = message.info.image;
  author.value = message.info.author;
  url.value = message.info.url;
  type.value = message.info.type;
  description.value = message.info.description;
  baseURL.value = message.info.baseURL;
  host.value = message.info.host;
});

onMounted(() => details());
</script>

<template>
  <div class="flex flex-col p-4 mx-auto my-2 shadow-2xl rounded-2xl">
    <h2 class="mb-2 text-lg font-bold place-self-center">SEO Preview</h2>

    <div class="border border-gray-300/60 w-[340px]">
      <div class="">
        <img
          v-if="isImageLoaded && image.startsWith('/')"
          v-show="isImageLoaded"
          :src="baseURL + image"
          alt="SEO image"
          class="w-[340px] h-[180px] object-cover object-center"
          @load="handleImageLoad"
        />
        <img
          v-else
          v-show="isImageLoaded"
          :src="image"
          alt="SEO image"
          class="w-[340px] h-[180px] object-cover object-center"
          @load="handleImageLoad"
        />
        <p v-show="!isImageLoaded" class="text-sm">Loading image...</p>
        <p v-show="!imageLoaded" class="text-sm">could not load image</p>
      </div>
      <div class="px-2 py-2 bg-slate-100/60">
        <p class="text-sm font-light uppercase line-clamp-1">
          {{ host }}
        </p>
        <p class="text-base font-semibold line-clamp-1">{{ title }}</p>
        <p class="text-sm font-light line-clamp-1">{{ description }}</p>
      </div>
    </div>

    <div class="pl-2 mt-2 text-sm">
      <h2 class="font-semibold">Other info</h2>
      <p class="ml-1"><strong>Author:</strong> {{ author }}</p>
      <p class="ml-1"><strong>Type:</strong> {{ type }}</p>
    </div>
    <a
      href="https://clenny.tech"
      class="mt-2 text-sm text-purple-600 underline place-self-center"
      target="_blank"
      >Get full overview</a
    >
  </div>
</template>

<style></style>
