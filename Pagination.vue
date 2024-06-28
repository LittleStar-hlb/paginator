<template>
  <div class="container">
    <div class="prev item" @click="prevClick">&lt;</div>
    <div class="first item" @click="firstClick" :class="{ active: activePage === firstData }">{{ firstData }}</div>
    <div class="more item" @click="prevMoreClick" v-show="startIndex > 0">...</div>
    <div class="list" @click="clickTarget" ref="list_dom">
      <div
        class="item"
        v-for="page in visiblePage"
        :key="page"
        :class="{ active: activePage === page }"
      >
        {{ page }}
      </div>
    </div>
    <div class="more item" @click="nextMoreClick" v-show="endIndex < datas.length">...</div>
    <div class="last item" @click="lastClick" :class="{ active: activePage === lastData }">{{ lastData }}</div>
    <div class="next item" @click="nextClick">&gt;</div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, Ref } from "vue";

const props = defineProps({
  pageCount: {
    type: Number,
    required: true,
  },
  pagerCount: {
    type: Number,
    default: 7,
  },
});
const emits = defineEmits(["currentPage"]);

const datas = new Array(Math.ceil(props.pageCount))
  .fill(1)
  .map((item, index) => item + index);

let visibleCount: number = 0;
let screenSize: number = 0;

const itemSize: number = 30;
const lastData: number = datas.pop();
const firstData: number = datas.shift();

const activePage: Ref<number> = ref(0);
const startIndex: Ref<number> = ref(0);
const endIndex: Ref<number> = ref(0);

const list_dom = ref<HTMLElement | null>(null);

const visiblePage = computed(() => datas.slice(startIndex.value, endIndex.value));

const prevClick = () => {
  const currentPage = activePage.value - 1;
  if (currentPage >= firstData) {
    const leftPointer = visibleCount;
    const rightPointer = datas.length - visibleCount + 2;

    startIndex.value =
      currentPage > leftPointer
        ? currentPage <= rightPointer
          ? startIndex.value - 1
          : startIndex.value
        : 0;

    endIndex.value = startIndex.value + visibleCount;
    activePage.value = currentPage;
    emits("currentPage", activePage.value);
  }
};

const nextClick = () => {
  const currentPage = activePage.value + 1;
  if (currentPage <= lastData) {
    const leftPointer = visibleCount;
    const rightPointer = datas.length - visibleCount + 2;

    endIndex.value =
      currentPage <= rightPointer
        ? currentPage > leftPointer
          ? endIndex.value + 1
          : endIndex.value
        : datas.length;

    startIndex.value = endIndex.value - visibleCount;
    activePage.value = currentPage;
    emits("currentPage", activePage.value);
  }
};

const clickTarget = (event: MouseEvent) => {
  const target = event.target as HTMLElement;
  if (target !== list_dom.value) {
    activePage.value = Number(target.textContent);
    emits("currentPage", activePage.value);
  }
};

const prevMoreClick = () => {
  startIndex.value = Math.max(0, startIndex.value - visibleCount);
  endIndex.value = startIndex.value + visibleCount;
  activePage.value = endIndex.value + 1;
  emits("currentPage", activePage.value);
};

const nextMoreClick = () => {
  startIndex.value = Math.min(endIndex.value, datas.length - visibleCount);
  endIndex.value = startIndex.value + visibleCount;
  activePage.value = startIndex.value + 2;
  emits("currentPage", activePage.value);
};

const firstClick = () => {
  startIndex.value = 0;
  endIndex.value = startIndex.value + visibleCount;
  activePage.value = firstData;
  emits("currentPage", activePage.value);
};

const lastClick = () => {
  startIndex.value = datas.length - visibleCount;
  endIndex.value = startIndex.value + visibleCount;
  activePage.value = lastData;
  emits("currentPage", activePage.value);
};

onMounted(() => {
  if (list_dom.value) {
    screenSize = itemSize * props.pagerCount;
    visibleCount = Math.floor(screenSize / itemSize) - 2;
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount;
    activePage.value = firstData;
    emits("currentPage", activePage.value);
  }
});
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  column-gap: 5px;
  width: auto;
  height: 30px;
  font-size: 14px;
}

.list {
  display: flex;
  column-gap: 5px;
  width: auto;
  height: inherit;
  overflow: hidden;
  box-sizing: border-box;
}

.item {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0 4px;
  width: auto;
  min-width: 30px;
  height: inherit;
  line-height: 30px;
  cursor: pointer;
  user-select: none;
  border: 1px solid #000;
  box-sizing: border-box;
  border-radius: 3px;
}

.active {
  background-color: #000;
  color: #fff;
}
</style>
