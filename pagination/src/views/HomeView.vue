<script setup lang="ts">
import { ref } from "vue";
import { useQuery, keepPreviousData } from "@tanstack/vue-query";
import type { Ref } from "vue";


const peopleFetcher = async (page: Ref<number>) => {
  const response = await fetch(
    `https://randomuser.me/api/?page=${page.value}&results=20&seed=abc`
  );
  const data = await response.json();

  // fake delay
  await new Promise((resolve) => {
    setTimeout(() => resolve(true), 1000);
  });
  return data?.results || [];
};

const page = ref(1);
const { isPending, isError, data, error, isFetching, isPlaceholderData } = useQuery({
  queryKey: ["people", page],
  queryFn: () => peopleFetcher(page),
  placeholderData: keepPreviousData
});

const nextPage = () => {
  if (!isPlaceholderData.value) {
    page.value = page.value + 1;
  }
};

const prevPage = () => {
  page.value = Math.max(page.value - 1, 1);
};

</script>

<template>
  <template v-if="!data || isPlaceholderData">
    <div class="pending">Loading...</div>
  </template>
  <template v-else>
    <div>
      <p>Current Page: {{ page }}</p>
      <button @click="prevPage" :disabled="isPlaceholderData || page === 1">
        {{ isPlaceholderData ? "Loading..." : "Prev Page" }}
      </button>
      <button @click="nextPage" :disabled="isPlaceholderData">
        {{ isPlaceholderData ? "Loading..." : "Next Page" }}
      </button>
    </div>
    <div class="data">
      <p v-for="item in data" :key="item.login.uuid">
        <span>{{ item.email }}</span>
      </p>
    </div>
  </template>
</template>

<style scoped>
.pending {
  position: relative;
  top: -250px;
}

.data {
  border: 1px solid var(--color-border);
}
</style>