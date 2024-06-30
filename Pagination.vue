<template>
  <div class="container">
    <div class="prev item" @click="prevEvent">&lt;</div>
    <div class="first item" @click="firstEvent" :class="{ active: activePage === firstData }">{{ firstData }}</div>
    <div class="more item" @click="prevMoreEvent" v-show="startIndex > 0">...</div>
    <div class="list" @click="clickTarget">
      <div
          class="item"
          v-for="page in visiblePage"
          :key="page"
          :class="{ active: activePage === page }"
      >
        {{ page }}
      </div>
    </div>
    <div class="more item" @click="nextMoreEvent" v-show="endIndex < datas.length">...</div>
    <div class="last item" @click="lastEvent" :class="{ active: activePage === lastData }">{{ lastData }}</div>
    <div class="next item" @click="nextEvent">&gt;</div>
    <label for="search" class="search-label">Go to</label>
    <input
      id="search"
      type="number"
      title="搜索页码"
      autocomplete="off"
      :min="firstData"
      :max="lastData"
      :value="activePage"
      @change="searchEvent" />
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

const lastData: number = datas.pop();
const firstData: number = datas.shift();

const activePage: Ref<number> = ref(0);
const startIndex: Ref<number> = ref(0);
const endIndex: Ref<number> = ref(0);

const visiblePage = computed(() => datas.slice(startIndex.value, endIndex.value));

function setCurrentPage(currentPage: number) {
  if (currentPage <= 0) {
    return;
  }

  if (currentPage > props.pageCount) {
    return;
  }

  activePage.value = currentPage;
  emits("currentPage", activePage.value);
}

function searchEvent(event: Event) {
  const target = event.target as HTMLInputElement;
  const currentPage = Math.ceil(
    Math.min(lastData, Math.max(firstData, Number(target.value)))
  );

  const leftPointer = currentPage - visibleCount / 2;
  const rightPointer = currentPage + visibleCount / 2;

  if (leftPointer <= firstData) {
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount;
  } else if (rightPointer >= lastData) {
    endIndex.value = datas.length;
    startIndex.value = endIndex.value - visibleCount;
  } else {
    startIndex.value = leftPointer - 1;
    endIndex.value = startIndex.value + visibleCount;
  }

  target.value = currentPage.toString();
  setCurrentPage(currentPage);
}

const prevEvent = () => {
  const currentPage = activePage.value;
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
  }

  setCurrentPage(activePage.value - 1);
};

const nextEvent = () => {
  const currentPage = activePage.value;
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
  }

  setCurrentPage(activePage.value + 1);
};

const clickTarget = (event: MouseEvent) => {
  const target = event.target as HTMLElement;
  setCurrentPage(Number(target.textContent))
};

const prevMoreEvent = () => {
  startIndex.value = Math.max(0, startIndex.value - visibleCount);
  endIndex.value = startIndex.value + visibleCount;
  activePage.value -= activePage.value >= lastData ? visibleCount + 1 : visibleCount;

  setCurrentPage(activePage.value);
};

const nextMoreEvent = () => {
  startIndex.value = Math.min(endIndex.value, datas.length - visibleCount);
  endIndex.value = startIndex.value + visibleCount;
  activePage.value += activePage.value <= firstData ? visibleCount + 1 : visibleCount;

  setCurrentPage(activePage.value);
};

const firstEvent = () => {
  startIndex.value = 0;
  endIndex.value = startIndex.value + visibleCount;

  setCurrentPage(firstData);
};

const lastEvent = () => {
  startIndex.value = datas.length - visibleCount;
  endIndex.value = startIndex.value + visibleCount;

  setCurrentPage(lastData);
};

onMounted(() => {
    visibleCount = props.pagerCount - 2;
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount;

    setCurrentPage(firstData);
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
  -webkit-user-select: none;
  user-select: none;
  border: 1px solid #000;
  box-sizing: border-box;
  border-radius: 3px;
}

.active {
  background-color: #000;
  color: #fff;
}

.search-label[for="search"] {
  margin-left: 20px;
}

#search {
  padding: 0 4px;
  width: 50px;
  height: inherit;
  text-align: center;
  border-radius: 3px;
  box-sizing: border-box;
  border: 1px solid #000;
  outline: none;
  font-size: inherit;
  transition: all 0.2s ease-in-out;
  appearance: none;
}

#search:focus {
  box-shadow: inset 0 0 1px .5px #000;
  transition: all 0.2s ease-in-out;
}

#search::-webkit-outer-spin-button,
#search::-webkit-inner-spin-button {
    -webkit-appearance: none;
}
</style>
