<script setup lang="ts">
import { ref } from 'vue';
import LocationInput from './components/LocationInput.vue';
import SceneSelector from './components/SceneSelector.vue';
import ResultCard from './components/ResultCard.vue';
import AMapLoader from '@amap/amap-jsapi-loader';

// Types
interface Location {
  lng: number;
  lat: number;
}

interface Participant {
  id: number;
  name: string;
  location: Location | null;
  address: string;
}

interface Place {
  id: string;
  name: string;
  location: Location;
  address: string;
  type: string;
  distance?: number;
  photos?: any[];
  [key: string]: any;
}

// State
const participants = ref<Participant[]>([
  { id: 1, name: '参与者 1', location: null, address: '' },
  { id: 2, name: '参与者 2', location: null, address: '' }
]);
const selectedScenes = ref<string[]>([]);
// Removed unused map variable
const searchResults = ref<Place[]>([]);
const isSearching = ref(false);
const showResults = ref(false);

// Constants
const MAX_PARTICIPANTS = 10;

// Methods
const addParticipant = () => {
  if (participants.value.length < MAX_PARTICIPANTS) {
    participants.value.push({
      id: Date.now(),
      name: `参与者 ${participants.value.length + 1}`,
      location: null,
      address: ''
    });
  }
};

const removeParticipant = (index: number) => {
  if (participants.value.length > 2) {
    participants.value.splice(index, 1);
    // Renumber names if needed or just leave unique IDs
  }
};

const handleLocationSelect = (index: number, location: any) => {
  if (participants.value[index]) {
    participants.value[index].location = location.lnglat;
    participants.value[index].address = location.name;
  }
};

const findMeetingPoint = async () => {
  if (selectedScenes.value.length === 0) {
    alert('请至少选择一个场景类型');
    return;
  }
  
  const validParticipants = participants.value.filter((p): p is Participant & { location: Location } => p.location !== null);
  if (validParticipants.length < 2) {
    alert('请至少填入两个参与者的位置');
    return;
  }

  isSearching.value = true;
  
  // 1. Calculate midpoint
  let totalLat = 0;
  let totalLng = 0;
  validParticipants.forEach(p => {
    // p.location is guaranteed not null by filter type predicate
    totalLat += p.location.lat;
    totalLng += p.location.lng;
  });
  
  const midLat = totalLat / validParticipants.length;
  const midLng = totalLng / validParticipants.length;
  const center = [midLng, midLat]; // Amap uses [lng, lat]

  console.log('Calculated Center:', center);

  // 2. Search for places near center
  // We need to use AMap.PlaceSearch
  // This logic will be implemented in the map initialization or a separate helper
  await searchNearby(center, selectedScenes.value);
  
  isSearching.value = false;
  showResults.value = true;
};

const searchNearby = (center: number[], types: string[]) => {
  return new Promise<void>((resolve) => {
    if (!(window as any).AMap) return resolve();

    // Use (window as any).AMap or define global AMap
    const AMap = (window as any).AMap;

    AMap.plugin(["AMap.PlaceSearch"], function() {
      const placeSearch = new AMap.PlaceSearch({
        type: types.join('|'), 
        pageSize: 20,
        pageIndex: 1,
        extensions: 'all',
      });

      placeSearch.searchNearBy('', center, 2000, function(status: any, result: any) {
        if (status === 'complete' && result.info === 'OK') {
          searchResults.value = result.poiList.pois;
        } else {
          searchResults.value = [];
          console.log('Search failed or no results:', result);
        }
        resolve();
      });
    });
  });
};

// Initialize Map (lazy load)
const initMap = () => {
  const apiKey = import.meta.env.VITE_AMAP_KEY || "YOUR_AMAP_KEY_HERE";
  const securityCode = import.meta.env.VITE_AMAP_SECURITY_JS_CODE || "";

  if (apiKey === "YOUR_AMAP_KEY_HERE") {
    console.warn("Please set VITE_AMAP_KEY in .env file");
    // Optionally show a UI alert
  }
  
  if (securityCode) {
    (window as any)._AMapSecurityConfig = {
      securityJsCode: securityCode,
    };
  }

  AMapLoader.load({
    key: apiKey, 
    version: "2.0",
    plugins: ['AMap.PlaceSearch', 'AMap.AutoComplete'],
  }).then((AMap) => {
    (window as any).AMap = AMap;
    // Map instance is not strictly needed for just search, but good for visualization
    // We can initialize a hidden map or just use the API services
    // For visualization, we will render a map in the result section
  }).catch(e => {
    console.error(e);
  });
};

initMap();

</script>

<template>
  <div class="min-h-screen bg-slate-50 font-sans text-slate-800">
    <!-- Header -->
    <header class="bg-white/80 backdrop-blur-md border-b border-white/20 sticky top-0 z-50 shadow-sm">
      <div class="max-w-4xl mx-auto px-4 py-4 flex items-center justify-center flex-col">
        <div class="flex items-center gap-2 mb-1">
          <div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center text-white font-bold">M</div>
          <h1 class="text-2xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-600 to-blue-400">MeetPoint</h1>
        </div>
        <p class="text-sm text-slate-500">智能会面点推荐 - 让每次聚会都找到完美地点</p>
      </div>
    </header>

    <main class="max-w-4xl mx-auto px-4 py-8 space-y-8">
      
      <!-- Input Section -->
      <section class="bg-white/70 backdrop-blur-md rounded-2xl p-6 shadow-lg border border-white/50">
        <h2 class="text-lg font-semibold mb-4 flex items-center gap-2">
          <span class="w-1 h-6 bg-blue-500 rounded-full"></span>
          参与者位置
        </h2>
        
        <div class="space-y-4">
          <LocationInput 
            v-for="(participant, index) in participants" 
            :key="participant.id"
            :participant="participant"
            :index="index"
            @remove="removeParticipant(index)"
            @select="handleLocationSelect(index, $event)"
            :can-remove="participants.length > 2"
          />
        </div>

        <button 
          v-if="participants.length < MAX_PARTICIPANTS"
          @click="addParticipant"
          class="mt-4 w-full py-3 border-2 border-dashed border-blue-200 rounded-xl text-blue-500 hover:bg-blue-50 hover:border-blue-300 transition-colors flex items-center justify-center gap-2 font-medium"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
          </svg>
          添加参与者
        </button>
      </section>

      <!-- Scene Selection -->
      <section class="bg-white/70 backdrop-blur-md rounded-2xl p-6 shadow-lg border border-white/50">
        <h2 class="text-lg font-semibold mb-4 flex items-center gap-2">
          <span class="w-1 h-6 bg-orange-500 rounded-full"></span>
          选择聚会场景 (1-3项)
        </h2>
        <SceneSelector v-model="selectedScenes" />
      </section>

      <!-- Action Button -->
      <div class="flex justify-center">
        <button 
          @click="findMeetingPoint"
          :disabled="isSearching"
          class="px-8 py-4 bg-gradient-to-r from-blue-600 to-blue-500 text-white rounded-full font-bold text-lg shadow-xl shadow-blue-200 hover:shadow-blue-300 transform hover:-translate-y-1 transition-all disabled:opacity-70 disabled:cursor-not-allowed"
        >
          {{ isSearching ? '正在计算中点...' : '查找最佳会面点' }}
        </button>
      </div>

      <!-- Results Section -->
      <section v-if="showResults" class="space-y-6 animate-fade-in">
        <h2 class="text-2xl font-bold text-center">推荐地点</h2>
        
        <div v-if="searchResults.length === 0" class="text-center text-slate-500 py-8">
          未找到合适的中点场所，请尝试更换场景或调整位置。
        </div>

        <div class="grid md:grid-cols-2 gap-4">
          <ResultCard 
            v-for="place in searchResults" 
            :key="place.id" 
            :place="place" 
          />
        </div>
      </section>

    </main>
  </div>
</template>

<style>
.animate-fade-in {
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
