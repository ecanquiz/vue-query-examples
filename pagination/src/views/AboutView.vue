<script setup lang="ts">
import { useInfiniteQuery } from "@tanstack/vue-query";
import type { UseInfiniteQueryOptions } from "@tanstack/vue-query";

const peopleFetcher = async ({ pageParam = 1 }) => {
  const response = await fetch(
    `https://randomuser.me/api/?page=${pageParam}&results=10&seed=abc`
  );
  const data = await response.json();

  // fake delay
  await new Promise((resolve) => {
    setTimeout(() => resolve(true), 1000);
  });

  // sent the cursor/page value and the results
  // set max to 3 pages of data
  return {
    pageData: data?.results || [],
    cursor: pageParam === 3 ? undefined : pageParam + 1,
  };
};

const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetching,
  isPending
} = useInfiniteQuery<UseInfiniteQueryOptions>({
  queryKey: ["people"],
  //@ts-ignore
  queryFn: peopleFetcher,
  getNextPageParam: (lastPage: Record<string, any>) => lastPage.cursor
});

const nextPage = () => {
  fetchNextPage();
};
</script>

<template>
  <div class="">
    <h1 v-if="!data">Loading...</h1>
  <button @click="nextPage" :disabled="isPending" v-if="hasNextPage">
    {{ isFetching ? "Loading..." : "Load More Data" }}
  </button>
  <div>
    <div v-for="(page, index) in data?.pages" :key="index">
      <div v-for="item in (page as unknown as Record<string, any>).pageData" :key="item.login.uuid">
        <p>{{ item.email }}</p>
      </div>
    </div>
  </div>
  </div>
</template>
