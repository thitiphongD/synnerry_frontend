<template>
  <div>
    <h1>
      Visit Counter <span>{{ counter }}</span>
    </h1>
    <p>Device: {{ device }}</p>
    <p>Browser: {{ browser }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const counter = ref(0);
const device = ref("");
const browser = ref("");

onMounted(() => {
  // Retrieve counter value from localStorage
  const storedCounter = localStorage.getItem("visitCounter");

  // If a value exists in localStorage, use it; otherwise, initialize to 0
  counter.value = storedCounter ? parseInt(storedCounter, 10) : 0;

  // Increment the counter when the component is mounted (page is visited)
  counter.value += 1;

  // Save the updated counter value to localStorage
  localStorage.setItem("visitCounter", counter.value.toString());

  // Get device information
  device.value = getDevice();

  // Get browser information
  browser.value = getBrowser();
});

// Function to get device information
const getDevice = () => {
  const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
  return isMobile ? "Mobile" : "Desktop";
};

// Function to get browser information
const getBrowser = () => {
  const userAgent = navigator.userAgent;
  if (userAgent.includes("Chrome")) return "Google Chrome";
  if (userAgent.includes("Safari")) return "Safari";
  if (userAgent.includes("Firefox")) return "Mozilla Firefox";
  if (userAgent.includes("Edge")) return "Microsoft Edge";
  if (userAgent.includes("Opera") || userAgent.includes("OPR")) return "Opera";
  return "Unknown";
};
</script>
