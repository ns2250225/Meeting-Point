<script setup lang="ts">
import { ref } from 'vue';
import html2canvas from 'html2canvas';

const props = defineProps<{
  place: any;
}>();

const isGenerating = ref(false);

const blobFromCanvas = (canvas: HTMLCanvasElement) =>
  new Promise<Blob>((resolve, reject) => {
    canvas.toBlob((b) => (b ? resolve(b) : reject(new Error('Blob creation failed'))), 'image/png');
  });

const writeImageToClipboard = async (blob: Blob) => {
  const ClipboardItemCtor = (window as any).ClipboardItem;
  if (!navigator.clipboard?.write || !ClipboardItemCtor) {
    throw new Error('Clipboard image write is not supported');
  }
  await navigator.clipboard.write([new ClipboardItemCtor({ 'image/png': blob })]);
};

const downloadBlob = (blob: Blob, filename: string) => {
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.download = filename;
  link.href = url;
  link.click();
  URL.revokeObjectURL(url);
};

const buildShareCardElement = () => {
  const mount = document.createElement('div');
  mount.style.position = 'fixed';
  mount.style.left = '-10000px';
  mount.style.top = '0';
  mount.style.zIndex = '-1';

  const card = document.createElement('div');
  card.style.all = 'initial';
  card.style.boxSizing = 'border-box';
  card.style.width = '360px';
  card.style.backgroundColor = '#ffffff';
  card.style.border = '1px solid #e2e8f0';
  card.style.borderRadius = '16px';
  card.style.overflow = 'hidden';
  card.style.fontFamily = "Inter, system-ui, -apple-system, 'Segoe UI', Arial, sans-serif";
  card.style.color = '#0f172a';

  const header = document.createElement('div');
  header.style.display = 'flex';
  header.style.alignItems = 'center';
  header.style.gap = '10px';
  header.style.padding = '14px 14px 10px 14px';
  header.style.borderBottom = '1px solid #f1f5f9';

  const logo = document.createElement('div');
  logo.style.width = '32px';
  logo.style.height = '32px';
  logo.style.borderRadius = '10px';
  logo.style.display = 'flex';
  logo.style.alignItems = 'center';
  logo.style.justifyContent = 'center';
  logo.style.backgroundColor = '#2563eb';
  logo.style.color = '#ffffff';
  logo.style.fontWeight = '800';
  logo.style.fontSize = '16px';
  logo.textContent = 'M';

  const headerText = document.createElement('div');
  headerText.style.display = 'flex';
  headerText.style.flexDirection = 'column';
  headerText.style.gap = '2px';

  const title = document.createElement('div');
  title.style.fontWeight = '800';
  title.style.fontSize = '14px';
  title.textContent = 'MeetPoint 推荐会面点';

  const subtitle = document.createElement('div');
  subtitle.style.fontSize = '11px';
  subtitle.style.color = '#64748b';
  subtitle.textContent = '智能会面点推荐 - 让每次聚会都找到完美地点';

  headerText.appendChild(title);
  headerText.appendChild(subtitle);

  header.appendChild(logo);
  header.appendChild(headerText);

  const body = document.createElement('div');
  body.style.padding = '14px';

  const placeName = document.createElement('div');
  placeName.style.fontSize = '18px';
  placeName.style.fontWeight = '800';
  placeName.style.lineHeight = '1.25';
  placeName.textContent = props.place?.name || '推荐地点';

  const placeAddress = document.createElement('div');
  placeAddress.style.marginTop = '8px';
  placeAddress.style.fontSize = '12px';
  placeAddress.style.color = '#475569';
  placeAddress.style.lineHeight = '1.4';
  placeAddress.textContent = props.place?.address || '暂无详细地址';

  const metaRow = document.createElement('div');
  metaRow.style.display = 'flex';
  metaRow.style.alignItems = 'center';
  metaRow.style.justifyContent = 'space-between';
  metaRow.style.marginTop = '12px';
  metaRow.style.paddingTop = '12px';
  metaRow.style.borderTop = '1px solid #f1f5f9';

  const tag = document.createElement('div');
  tag.style.display = 'inline-flex';
  tag.style.alignItems = 'center';
  tag.style.padding = '6px 10px';
  tag.style.borderRadius = '999px';
  tag.style.backgroundColor = '#dbeafe';
  tag.style.color = '#2563eb';
  tag.style.fontWeight = '700';
  tag.style.fontSize = '11px';
  const typeText = (props.place?.type || '').split(';')[0].split('|')[0] || '场景';
  tag.textContent = typeText;

  const distance = document.createElement('div');
  distance.style.fontSize = '11px';
  distance.style.color = '#94a3b8';
  distance.textContent = `距离中点：${props.place?.distance ? `${props.place.distance}米` : '附近'}`;

  metaRow.appendChild(tag);
  metaRow.appendChild(distance);

  const footer = document.createElement('div');
  footer.style.marginTop = '12px';
  footer.style.fontSize = '10px';
  footer.style.color = '#94a3b8';
  footer.textContent = '复制自 MeetPoint';

  body.appendChild(placeName);
  body.appendChild(placeAddress);
  body.appendChild(metaRow);
  body.appendChild(footer);

  card.appendChild(header);
  card.appendChild(body);

  mount.appendChild(card);
  document.body.appendChild(mount);
  return { mount, card };
};

const handleShare = async () => {
  isGenerating.value = true;
  const { mount, card } = buildShareCardElement();
  try {
    const canvas = await html2canvas(card, { backgroundColor: '#ffffff', scale: 2, useCORS: false });
    const blob = await blobFromCanvas(canvas);
    await writeImageToClipboard(blob);
    alert('卡片图片已复制到剪贴板！');
  } catch (e: any) {
    try {
      const canvas = await html2canvas(card, { backgroundColor: '#ffffff', scale: 2, useCORS: false });
      const blob = await blobFromCanvas(canvas);
      downloadBlob(blob, `meetpoint-${props.place?.name || 'card'}.png`);
      alert('复制失败，已为您下载图片。');
    } catch (err: any) {
      alert(`生成图片出错: ${err?.message || e?.message || '未知错误'}`);
    }
  } finally {
    mount.remove();
    isGenerating.value = false;
  }
};
</script>

<template>
  <div class="bg-white rounded-xl overflow-hidden shadow-lg border border-slate-100 hover:shadow-xl transition-shadow group flex flex-col h-full">
    <div class="h-32 bg-slate-100 relative overflow-hidden shrink-0">
      <div class="absolute inset-0 bg-gradient-to-br from-blue-50 to-indigo-50 flex items-center justify-center text-slate-300">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
        </svg>
      </div>
    </div>
    
    <div class="p-4 flex flex-col flex-1">
      <div class="flex justify-between items-start mb-2">
        <h3 class="text-lg font-bold text-slate-800 line-clamp-1" :title="place.name">{{ place.name }}</h3>
        <span class="text-xs font-semibold px-2 py-1 bg-blue-100 text-blue-600 rounded-full shrink-0 ml-2">
          {{ place.type.split(';')[0].split('|')[0] }}
        </span>
      </div>
      
      <p class="text-sm text-slate-500 mb-3 line-clamp-2 h-10" :title="place.address">
        {{ place.address || '暂无详细地址' }}
      </p>
      
      <div class="mt-auto pt-3 border-t border-slate-50 flex items-center justify-between">
        <div class="text-xs text-slate-400">
          距离中点: {{ place.distance ? place.distance + '米' : '附近' }}
        </div>
        
        <button 
          @click="handleShare"
          :disabled="isGenerating"
          data-html2canvas-ignore="true"
          class="flex items-center gap-1 text-sm font-medium text-blue-600 hover:text-blue-700 hover:bg-blue-50 px-3 py-1.5 rounded-lg transition-colors disabled:opacity-50 disabled:cursor-wait"
        >
          <svg v-if="!isGenerating" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
            <path d="M15 8a3 3 0 10-2.977-2.63l-4.94 2.47a3 3 0 100 4.319l4.94 2.47a3 3 0 10.895-1.789l-4.94-2.47a3.027 3.027 0 000-.74l4.94-2.47C13.456 7.68 14.19 8 15 8z" />
          </svg>
          <span v-else class="animate-spin h-4 w-4 border-2 border-blue-600 border-t-transparent rounded-full mr-1"></span>
          {{ isGenerating ? '生成中...' : '复制卡片图片' }}
        </button>
      </div>
    </div>
  </div>
</template>
