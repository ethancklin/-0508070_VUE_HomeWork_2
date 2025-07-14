<script setup>
import { ref, computed, onMounted } from 'vue';

// 定義資料和狀態
const stations = ref([]);
const searchKeyword = ref('');
const selectedStation = ref(null);
const favorites = ref([]);

// 載入收藏
const loadFavorites = () => {
  const stored = localStorage.getItem('youbike_favorites');
  if (stored) {
    try {
      favorites.value = JSON.parse(stored);
    } catch (e) {
      favorites.value = [];
    }
  }
};

// 儲存收藏
const saveFavorites = () => {
  localStorage.setItem('youbike_favorites', JSON.stringify(favorites.value));
};

// 取得資料
const fetchData = async () => {
  try {
    const response = await fetch(
      'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
    );
    const data = await response.json();
    stations.value = data;
  } catch (error) {
    console.error('資料載入失敗', error);
  }
};

// 觸發資料載入
onMounted(() => {
  loadFavorites();
  fetchData();
});

// 計算過濾後的站點
const filteredStations = computed(() => {
  if (!searchKeyword.value) return [];
  const keyword = searchKeyword.value.trim().toLowerCase();
  return stations.value.filter(
    (s) =>
      (s.sna && s.sna.toLowerCase().includes(keyword)) ||
      (s.ar && s.ar.toLowerCase().includes(keyword))
  );
});

// 搜尋站點
const searchStations = () => {
  if (filteredStations.value.length > 0) {
    selectedStation.value = filteredStations.value[0];
  } else {
    selectedStation.value = null;
  }
};

// 選擇站點
const selectStation = (station) => {
  selectedStation.value = station;
};

// 收藏/取消收藏
const toggleFavorite = (station) => {
  const index = favorites.value.findIndex((fav) => fav.sno === station.sno);
  if (index === -1) {
    favorites.value.push(station);
  } else {
    favorites.value.splice(index, 1);
  }
  saveFavorites();
};

// 是否已收藏
const isFavorited = (station) => {
  return favorites.value.some((fav) => fav.sno === station.sno);
};
</script>

<template class="container">
  <div class="flex flex-col items-center min-h-screen p-4">
    <h1 class="text-5xl flex space-x-2 mb-5">台北市 Youbike 站點查詢</h1>
    <!-- 搜索區域 -->
    <div class="flex items-center mb-5">
      <input
        type="text"
        class="input mb-5 bg-size-2"
        v-model.trim="searchKeyword"
        @keyup.enter="searchStations"
        placeholder="請輸入站點名稱或地址"
      />
      <button @click="searchStations" class="btn btn-warning btn-sl mb-5 mx-3">
        搜尋
      </button>
    </div>

    <!-- 查詢結果列表 -->
    <div v-if="filteredStations.length">
      <h2
        class="text-1xl mb-5 text-blue-600 dark:text-sky-400 border-4 border-indigo-200"
      >
        搜尋結果
      </h2>
      <ul>
        <li
          v-for="station in filteredStations"
          :key="station.sno"
          @click="selectStation(station)"
          class="text-1xl mb-2 text-dark-600 dark:text-sky-400 hover:text-blue-500 cursor-pointer"
        >
          {{ station.sarea }} - {{ station.sna }} [地址--<span
            class="text-red-500"
            >{{ station.ar }}</span
          >]
        </li>
      </ul>
    </div>
    <div v-else-if="searchKeyword">
      <p>未找到符合條件的站點</p>
    </div>

    <!-- 詳細資訊 -->
    <div
      v-if="selectedStation"
      style="margin-top: 20px; border: 2px solid #000; padding: 10px"
    >
      <h3 class="text-1xl mb-2 text-blue-600 text-center">站點詳細資訊</h3>
      <p class="mb-2"><strong>站名：</strong> {{ selectedStation.sna }}</p>
      <p class="mb-2"><strong>地址：</strong> {{ selectedStation.ar }}</p>
      <p class="mb-2">
        <strong>剩餘自行車數：</strong>
        {{ selectedStation.available_rent_bikes }}
      </p>
      <p class="mb-2">
        <strong>總共車格：</strong> {{ selectedStation.Quantity }}
      </p>
      <button
        @click="toggleFavorite(selectedStation)"
        class="btn btn-error btn-sm ml-auto"
      >
        {{ isFavorited(selectedStation) ? '刪除收藏' : '加入收藏' }}
      </button>
    </div>

    <!-- 收藏站點 -->
    <div>
      <h2
        class="text-1xl mb-5 mt-5 text-yellow-600 dark:text-sky-400 border-4 border-2 border-4 border-double text-center"
      >
        我的收藏
      </h2>
      <ul>
        <table
          class="border-separate border-spacing-2 border border-gray-400 dark:border-gray-500"
        >
          <thead>
            <tr>
              <th class="border border-gray-300 dark:border-gray-600">區域</th>
              <th class="border border-gray-300 dark:border-gray-600">站點</th>
              <th class="border border-gray-300 dark:border-gray-600">地址</th>
            </tr>
          </thead>
          <tbody>
            <tr
              class="border border-gray-300 dark:border-gray-600 hover:text-blue-500 cursor-pointer"
              v-for="fav in favorites"
              :key="fav.sno"
              @click="selectStation(fav)"
            >
              <td
                class="border border-gray-300 dark:border-gray-700 text-center"
              >
                {{ fav.sarea }}
              </td>
              <td
                class="border border-gray-300 dark:border-gray-700 text-center"
              >
                {{ fav.sna }}
              </td>
              <td
                class="border border-gray-300 dark:border-gray-700 text-center"
              >
                {{ fav.ar }}
              </td>
            </tr>
          </tbody>
        </table>
      </ul>
    </div>
  </div>
</template>

<style>
/* 簡單樣式，可自定義擴充 */
</style>
