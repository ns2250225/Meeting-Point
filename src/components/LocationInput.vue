<script setup lang="ts">
import { ref, watch } from 'vue';

const props = defineProps<{
  participant: { id: number; name: string; location: any; address: string };
  index: number;
  canRemove: boolean;
}>();

const emit = defineEmits(['remove', 'select']);

const searchQuery = ref('');
const searchResults = ref<any[]>([]);
const isSearching = ref(false);
const showDropdown = ref(false);

const handleSearch = () => {
  if (!searchQuery.value.trim()) return;
  if (!window.AMap) {
    alert('地图组件尚未加载完成，请稍候');
    return;
  }

  isSearching.value = true;
  
  AMap.plugin('AMap.AutoComplete', function(){
    // 实例化AutoComplete
    const autoOptions = {
      city: '全国'
    }
    const autoComplete = new AMap.AutoComplete(autoOptions);
    autoComplete.search(searchQuery.value, function(status: any, result: any) {
      isSearching.value = false;
      if (status === 'complete' && result.info === 'OK') {
        searchResults.value = result.tips;
        showDropdown.value = true;
      } else {
        searchResults.value = [];
        showDropdown.value = false;
        // Fallback to PlaceSearch if AutoComplete returns nothing or for more details
        // But Tips usually works for input
      }
    })
  })
};

const selectLocation = (tip: any) => {
  if (!tip.location) {
    alert('该地点没有坐标信息，请选择其他地点');
    return;
  }
  searchQuery.value = tip.name;
  showDropdown.value = false;
  emit('select', {
    name: tip.name,
    lnglat: tip.location, // AMap.LngLat object
    address: tip.district + tip.address
  });
};

// Close dropdown when clicking outside (simplified)
// In a real app, use a directive or event listener on window
</script>

<template>
  <div class="relative bg-white/50 rounded-xl p-4 border border-white/60 shadow-sm transition-all hover:shadow-md">
    <div class="flex items-center justify-between mb-2">
      <label class="text-sm font-medium text-slate-600">{{ participant.name }}</label>
      <button 
        v-if="canRemove" 
        @click="$emit('remove')" 
        class="text-red-400 hover:text-red-600 text-sm transition-colors"
      >
        移除
      </button>
    </div>
    
    <div class="flex gap-2 relative">
      <div class="relative flex-1">
        <input 
          v-model="searchQuery"
          @keyup.enter="handleSearch"
          type="text" 
          placeholder="输入地址或地标..." 
          class="w-full px-4 py-2.5 rounded-lg border border-slate-200 bg-white focus:ring-2 focus:ring-blue-400 focus:border-transparent outline-none transition-all"
        />
        
        <!-- Dropdown Results -->
        <div v-if="showDropdown && searchResults.length > 0" class="absolute top-full left-0 right-0 mt-1 bg-white rounded-lg shadow-xl border border-slate-100 z-20 max-h-60 overflow-y-auto">
          <ul>
            <li 
              v-for="(tip, i) in searchResults" 
              :key="i"
              @click="selectLocation(tip)"
              class="px-4 py-3 hover:bg-blue-50 cursor-pointer border-b border-slate-50 last:border-none"
            >
              <div class="font-medium text-slate-800">{{ tip.name }}</div>
              <div class="text-xs text-slate-400 truncate">{{ tip.district }}{{ tip.address }}</div>
            </li>
          </ul>
        </div>
      </div>

      <button 
        @click="handleSearch"
        class="px-4 py-2 bg-blue-100 text-blue-600 rounded-lg hover:bg-blue-200 transition-colors font-medium flex items-center"
      >
        <span v-if="isSearching" class="animate-spin mr-1">↻</span>
        搜索
      </button>
    </div>
    
    <div v-if="participant.location" class="mt-2 text-xs text-green-600 flex items-center gap-1">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" viewBox="0 0 20 20" fill="currentColor">
        <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
      </svg>
      已选择: {{ participant.address }}
    </div>
  </div>
</template>
