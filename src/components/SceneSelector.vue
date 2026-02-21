<script setup lang="ts">
const props = defineProps<{
  modelValue: string[];
}>();

const emit = defineEmits(['update:modelValue']);

const scenes = [
  '咖啡馆', '餐厅', '酒吧', '茶楼', '茶馆', 
  '电影院', 'KTV', '游戏厅', '健身房', 
  '图书馆', '书店', '博物馆', '景点', '商场', '公园'
];

const toggleScene = (scene: string) => {
  const newSelection = [...props.modelValue];
  const index = newSelection.indexOf(scene);
  
  if (index > -1) {
    newSelection.splice(index, 1);
  } else {
    if (newSelection.length < 3) {
      newSelection.push(scene);
    } else {
      // Optional: alert or visual feedback for max limit
      // For now, just ignore
    }
  }
  
  emit('update:modelValue', newSelection);
};
</script>

<template>
  <div class="grid grid-cols-3 sm:grid-cols-4 md:grid-cols-5 gap-3">
    <button 
      v-for="scene in scenes" 
      :key="scene"
      @click="toggleScene(scene)"
      class="px-3 py-2 rounded-lg text-sm font-medium transition-all duration-200 border"
      :class="modelValue.includes(scene) 
        ? 'bg-orange-100 border-orange-300 text-orange-700 shadow-sm ring-2 ring-orange-200 ring-offset-1' 
        : 'bg-white border-slate-200 text-slate-600 hover:border-slate-300 hover:bg-slate-50'"
    >
      {{ scene }}
    </button>
  </div>
  <div class="mt-2 text-xs text-slate-400 text-right">
    已选: {{ modelValue.length }}/3
  </div>
</template>
