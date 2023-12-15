<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-semibold mb-6">Visit Counter: {{ counter }}</h1>
    <div class="flex gap-2">
      <h1 class="text-3xl font-semibold mb-6">IP Address: {{ ipAddress }}</h1>
      <h1 class="text-3xl font-semibold mb-6">Device: {{ device }}</h1>
      <h1 class="text-3xl font-semibold mb-6">Browser: {{ browser }}</h1>
    </div>

    <div class="overflow-x-auto">
      <table
        class="min-w-full bg-white border border-gray-300 rounded-lg shadow-md"
      >
        <thead>
          <tr class="bg-gray-200 text-gray-700">
            <th
              @click="sortBy('ipAddress')"
              class="py-3 px-6 border-b border-gray-300 cursor-pointer"
            >
              IP Address
            </th>
            <th
              @click="sortBy('device')"
              class="py-3 px-6 border-b border-gray-300 cursor-pointer"
            >
              Device
            </th>
            <th
              @click="sortBy('browser')"
              class="py-3 px-6 border-b border-gray-300 cursor-pointer"
            >
              Browser
            </th>
            <th
              @click="sortBy('accessedAt')"
              class="py-3 px-6 border-b border-gray-300 cursor-pointer"
            >
              Accessed At
            </th>
            <th class="py-3 px-6 border-b border-gray-300">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="visit in apiData"
            :key="visit._id"
            class="hover:bg-gray-100"
          >
            <td class="py-4 px-6 border-b border-gray-300">
              {{ visit.ipAddress }}
            </td>
            <td class="py-4 px-6 border-b border-gray-300">
              {{ visit.device }}
            </td>
            <td class="py-4 px-6 border-b border-gray-300">
              {{ visit.browser }}
            </td>
            <td class="py-4 px-6 border-b border-gray-300">
              {{ visit.accessedAt }}
            </td>
            <td class="py-4 px-6 border-b border-gray-300">
              <button
                @click="deleteVisit(visit._id)"
                class="bg-red-500 text-white px-4 py-2 rounded"
              >
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";

const counter = ref(0);
const device = ref("");
const browser = ref("");
const ipAddress = ref("");
const apiData = ref([]);

onMounted(async () => {
  try {
    const response = await axios.get("http://localhost:8080/api/visitor/count");
    counter.value = response.data.visitorCount;
    apiData.value = response.data.visitors;
  } catch (error) {
    console.error("Error fetching data:", error);
  }
  device.value = getDevice();
  browser.value = getBrowser();
  ipAddress.value = await getIpAddress();
  saveVisitor();
});

const getDevice = () => {
  const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
  return isMobile ? "Mobile" : "Desktop";
};

const getBrowser = () => {
  const userAgent = navigator.userAgent;
  if (userAgent.includes("Chrome")) return "Google Chrome";
  if (userAgent.includes("Safari")) return "Safari";
  if (userAgent.includes("Firefox")) return "Mozilla Firefox";
  if (userAgent.includes("Edge")) return "Microsoft Edge";
  if (userAgent.includes("Opera") || userAgent.includes("OPR")) return "Opera";
  return "Unknown";
};

const saveVisitor = async () => {
  try {
    const visitorData = {
      ipAddress: ipAddress.value,
      device: device.value,
      browser: browser.value,
    };
    await axios.post("http://localhost:8080/api/visitor", visitorData);
  } catch (error) {
    console.error("Error saving visitor:", error);
  }
};

const getIpAddress = async () => {
  try {
    const response = await axios.get("https://api.ipify.org?format=json");
    return response.data.ip;
  } catch (error) {
    console.error("Error getting IP address:", error);
    return null;
  }
};

const deleteVisit = async (visitId: string) => {
  try {
    await axios.delete(`http://localhost:8080/api/visitor/${visitId}/delete`);

    apiData.value = apiData.value.filter((visit) => visit._id !== visitId);
    counter.value--;
  } catch (error) {
    console.error(`Error deleting visit with ID ${visitId}:`, error);
  }
};

const sorting = ref({
  key: "",
  order: "asc",
});

const sortBy = (key: string) => {
  if (sorting.value.key === key) {
    sorting.value.order = sorting.value.order === "asc" ? "desc" : "asc";
  } else {
    sorting.value.key = key;
    sorting.value.order = "asc";
  }

  apiData.value.sort((a, b) => {
    const aValue = a[key];
    const bValue = b[key];

    if (sorting.value.order === "asc") {
      return aValue.localeCompare(bValue);
    } else {
      return bValue.localeCompare(aValue);
    }
  });
};
</script>
