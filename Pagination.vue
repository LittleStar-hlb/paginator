<template>
  <div class="container">
    <div class="prev item" @click="prevEvent">&lt;</div>
    <div class="first item" @click="firstEvent" :class="{ active: activePage === firstData }">{{ firstData }}</div>

    <div class="more item" @click="prevMoreEvent" v-show="showPreMore">...</div>

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

    <div class="more item" @click="nextMoreEvent" v-show="showLastMore">...</div>

    <div class="last item" @click="lastEvent" :class="{ active: activePage == lastData }">{{ lastData }}</div>
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
import { ref, onMounted, Ref } from "vue";

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


let showLastMore = true;
let showPreMore = false;

const firstData: number = 1;
const lastData: number = props.pageCount;

const activePage: Ref<number> = ref(0);

let visiblePage: Array<number> = new Array(2);

// 页码为1，页码为n 是必有的
// 中间页码 = 先前页码 + 当前页码 + 后续页码。如果当前页码不是1和n时
// len(中间页码) = 显示页数 - 2
// 如果中间页码为偶数，去除掉当前页码，则 先前页码 + 1 = 后续页码
// 如果中间页码为奇数，去除掉当前页码，则 先前页码 = 后续页码

let middlePage = props.pagerCount - 2;
// 求得先前有多少个页码
let prePageLength = middlePage % 2 === 0 ? middlePage / 2 : (middlePage - 1) / 2;

// if (middlePage % 2 === 0) {
//   // prePage + prePage = middlePage
//   prePage = middlePage / 2;
// } else {
//   // prePage + prePage + 1 = middlePage
//   prePage = (middlePage - 1) / 2;
// }


function setCurrentPage(currentPage: number) {
  if (currentPage <= 0) {
    return;
  }

  if (currentPage > lastData) {
    return;
  }

  activePage.value = currentPage;
  emits("currentPage", activePage.value);

  doRefresh(currentPage);
}

function doRefresh(currentPage: number) {
  // 先前的页码
  let prePage : number = currentPage - prePageLength;

  showLastMore = true;
  showPreMore = true;

  // 如果先前的页码 小等于 1，则先前页码应为2开始
  if (prePage <= 2) {
    prePage = 2;
  }

  // 如果后续页码大等于 n，则先前页码 应为 总页数 - 中间页数
  if (prePage + middlePage >= lastData) {
    prePage = lastData - middlePage
  }

  // 生成输出序列
  visiblePage = Array.from(
      {length: middlePage},
      (_, i) => prePage + i
  )

  showPreMore = !(prePage === 2)
  showLastMore = !(prePage === lastData - middlePage);
}

function searchEvent(event: Event) {
  const target = event.target as HTMLInputElement;
  const targetNumber : number = Number(target.value);

  if (isNaN(targetNumber) || targetNumber < firstData || targetNumber > lastData) {
    target.value = String(activePage.value)
    return;
  }

  setCurrentPage(targetNumber);
}

const prevEvent = () => {
  setCurrentPage(activePage.value - 1);
};

const nextEvent = () => {
  setCurrentPage(activePage.value + 1);
};

const clickTarget = (event: MouseEvent) => {
  const target = event.target as HTMLElement;
  setCurrentPage(Number(target.textContent))
};

const prevMoreEvent = () => {
  setCurrentPage(activePage.value - 4);
};

const nextMoreEvent = () => {
  setCurrentPage(activePage.value + 4);
};

const firstEvent = () => {
  setCurrentPage(firstData);
};

const lastEvent = () => {
  setCurrentPage(lastData);
};

onMounted(() => {
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
