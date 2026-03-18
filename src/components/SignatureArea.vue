<script setup lang="ts">
import { ref, onMounted, onUnmounted, shallowRef } from 'vue';
import SignaturePad from 'signature_pad';
import { RefreshCcw, Undo2, Download } from 'lucide-vue-next';

const canvas = ref<HTMLCanvasElement | null>(null);
const signaturePad = shallowRef<SignaturePad | null>(null);
const outputFormat = ref<'png' | 'jpeg'>('png');
const hasSignature = ref(false);
const previewDataURL = ref<string | null>(null);

const updatePreview = () => {
  if (signaturePad.value && !signaturePad.value.isEmpty()) {
    previewDataURL.value = signaturePad.value.toDataURL(`image/${outputFormat.value}`);
  } else {
    previewDataURL.value = null;
  }
};

const initSignaturePad = () => {
  if (!canvas.value) return;
  signaturePad.value = new SignaturePad(canvas.value, {
    penColor: '#0f172a',
    backgroundColor: '#ffffff',
    minWidth: 0.5,
    maxWidth: 5,
    velocityFilterWeight: 0.7,
  });

  signaturePad.value.addEventListener('endStroke', () => {
    hasSignature.value = !signaturePad.value?.isEmpty();
    updatePreview();
  });
};

const resizeCanvas = () => {
  if (!canvas.value || !signaturePad.value) return;

  const ratio = Math.max(window.devicePixelRatio || 1, 1);
  const data = signaturePad.value.toData(); // Save existing drawing

  // Using parent properties to maintain responsiveness
  const parent = canvas.value.parentElement;
  if (parent) {
    canvas.value.width = parent.offsetWidth * ratio;
    canvas.value.height = parent.offsetHeight * ratio;
    
    // Scale context
    const ctx = canvas.value.getContext('2d');
    if (ctx) ctx.scale(ratio, ratio);
    
    // Restore drawing
    signaturePad.value.clear();
    signaturePad.value.fromData(data);
  }
};

const clearPad = () => {
  if (signaturePad.value) {
    signaturePad.value.clear();
    hasSignature.value = false;
    updatePreview();
  }
};

const undoLast = () => {
  if (signaturePad.value) {
    const data = signaturePad.value.toData();
    if (data && data.length > 0) {
      data.pop();
      signaturePad.value.fromData(data);
      hasSignature.value = !signaturePad.value.isEmpty();
      updatePreview();
    } else {
      clearPad();
    }
  }
};

const downloadSignature = () => {
  if (!signaturePad.value || signaturePad.value.isEmpty()) return;

  const dataURL = signaturePad.value.toDataURL(`image/${outputFormat.value}`);
  const a = document.createElement('a');
  a.href = dataURL;
  a.download = `signature.${outputFormat.value === 'jpeg' ? 'jpg' : 'png'}`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
};

onMounted(() => {
  initSignaturePad();
  resizeCanvas();
  window.addEventListener('resize', resizeCanvas);
});

import { watch } from 'vue';
watch(outputFormat, () => {
  updatePreview();
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeCanvas);
  signaturePad.value?.off();
});
</script>

<template>
  <div class="signature-container">
    <div class="pad-wrapper">
      <canvas ref="canvas" class="signature-canvas"></canvas>
      <div class="pad-placeholder" v-if="!hasSignature">
        <p>Sign here</p>
      </div>
    </div>
    
    <div class="preview-wrapper" v-if="hasSignature && previewDataURL">
      <h4>Output Preview</h4>
      <div class="preview-box">
        <img :src="previewDataURL" alt="Signature Preview" />
      </div>
    </div>
    
    <div class="controls">
      <div class="action-group">
        <button class="btn btn-secondary" @click="clearPad" :disabled="!hasSignature">
          <RefreshCcw :size="18" /> Cleanup
        </button>
        <button class="btn btn-secondary" @click="undoLast" :disabled="!hasSignature">
          <Undo2 :size="18" /> Undo
        </button>
      </div>
      
      <div class="export-group">
        <div class="format-toggle">
          <label :class="{ active: outputFormat === 'png' }">
            <input type="radio" v-model="outputFormat" value="png" />
            PNG
          </label>
          <label :class="{ active: outputFormat === 'jpeg' }">
            <input type="radio" v-model="outputFormat" value="jpeg" />
            JPG
          </label>
        </div>
        
        <button class="btn btn-primary" @click="downloadSignature" :disabled="!hasSignature">
          <Download :size="18" /> Export
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Scoped component styles */
.signature-container {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

.pad-wrapper {
  position: relative;
  width: 100%;
  height: 300px; /* Base height for mobile */
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 24px -6px rgba(0, 0, 0, 0.08), 0 0 1px 0 rgba(0,0,0,0.1);
  overflow: hidden;
  border: 1px solid var(--border-color);
  transition: box-shadow 0.3s ease;
}

.pad-wrapper:focus-within {
  box-shadow: 0 0 0 3px var(--primary-light), 0 4px 24px -6px rgba(0, 0, 0, 0.08);
}

.signature-canvas {
  width: 100%;
  height: 100%;
  touch-action: none;
  cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="%230f172a" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20h9"/><path d="M16.5 3.5a2.12 2.12 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>') 0 24, crosshair;
  display: block;
}

.pad-placeholder {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  opacity: 0.3;
  user-select: none;
}

.pad-placeholder p {
  font-size: 1.25rem;
  font-weight: 500;
  color: #64748b;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  background: white;
  padding: 1.25rem;
  border-radius: 16px;
  box-shadow: 0 4px 24px -6px rgba(0, 0, 0, 0.05);
  border: 1px solid var(--border-color);
}

.action-group, .export-group {
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.export-group {
  flex-wrap: wrap;
  justify-content: space-between;
  padding-top: 1rem;
  border-top: 1px solid var(--border-color);
}

.preview-wrapper {
  background: white;
  padding: 1rem;
  border-radius: 12px;
  box-shadow: 0 2px 12px -4px rgba(0, 0, 0, 0.05);
  border: 1px dashed var(--primary-light);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  animation: slideDown 0.3s ease-out;
}

@keyframes slideDown {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

.preview-wrapper h4 {
  font-size: 0.875rem;
  color: #64748b;
  margin: 0;
  text-align: center;
}

.preview-box {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100px;
  background: var(--bg-main);
  border-radius: 8px;
  overflow: hidden;
}

.preview-box img {
  max-width: 100%;
  max-height: 150px;
  object-fit: contain;
}

.format-toggle {
  display: flex;
  background: var(--bg-muted, #f1f5f9);
  padding: 0.25rem;
  border-radius: 8px;
  gap: 0.25rem;
}

.format-toggle label {
  cursor: pointer;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  color: #64748b;
  transition: all 0.2s ease;
  user-select: none;
}

.format-toggle input {
  display: none;
}

.format-toggle label.active {
  background: white;
  color: #0f172a;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 0.75rem 1.25rem;
  border-radius: 10px;
  font-weight: 600;
  font-size: 0.95rem;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  border: none;
  font-family: inherit;
  outline: none;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}

.btn:active:not(:disabled) {
  transform: scale(0.97);
}

.btn-secondary {
  background: #f1f5f9;
  color: #334155;
}

.btn-secondary:hover:not(:disabled) {
  background: #e2e8f0;
}

.btn-primary {
  background: var(--primary-color);
  color: white;
  flex: 1;
  min-width: 120px;
}

.btn-primary:hover:not(:disabled) {
  background: var(--primary-dark);
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
}

/* Responsive adjustments */
@media (min-width: 640px) {
  .pad-wrapper {
    height: 350px;
  }
  
  .controls {
    flex-direction: row;
    justify-content: space-between;
    padding: 1rem 1.5rem;
  }
  
  .export-group {
    padding-top: 0;
    border-top: none;
    flex-wrap: nowrap;
  }
  
  .action-group {
    flex: 1;
  }
  
  .btn-primary {
    flex: 0;
  }
}
</style>
