<template>
    <div class="custom-area-selector">
        <div class="preview-area" ref="previewArea">
            <div class="selection-area" ref="selectionArea" :style="selectionStyle"></div>
        </div>
        <div class="controls">
            <button @click="adjustSelection('top')" class="btn btn-sm btn-primary">Arriba</button>
            <button @click="adjustSelection('right')" class="btn btn-sm btn-primary">Derecha</button>
            <button @click="adjustSelection('bottom')" class="btn btn-sm btn-primary">Abajo</button>
            <button @click="adjustSelection('left')" class="btn btn-sm btn-primary">Izquierda</button>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const previewArea = ref(null);
const selectionArea = ref(null);

const selection = ref({
    x: 0,
    y: 0,
    width: 400,
    height: 300
});

const selectionStyle = computed(() => ({
    left: `${selection.value.x}px`,
    top: `${selection.value.y}px`,
    width: `${selection.value.width}px`,
    height: `${selection.value.height}px`
}));

function adjustSelection(direction) {
    const step = 10;
    switch (direction) {
        case 'top':
            selection.value.y = Math.max(selection.value.y - step, 0);
            break;
        case 'right':
            selection.value.x = Math.min(selection.value.x + step, previewArea.value.offsetWidth - selection.value.width);
            break;
        case 'bottom':
            selection.value.y = Math.min(selection.value.y + step, previewArea.value.offsetHeight - selection.value.height);
            break;
        case 'left':
            selection.value.x = Math.max(selection.value.x - step, 0);
            break;
    }
}
</script>

<style>
.custom-area-selector {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
}

.preview-area {
    position: relative;
    width: 100%;
    height: 300px;
    border: 1px solid #ccc;
    overflow: hidden;
}

.selection-area {
    position: absolute;
    border: 2px solid #007bff;
    background-color: rgba(0, 123, 255, 0.2);
    cursor: move;
}

.controls {
    display: flex;
    gap: 0.5rem;
}
</style>