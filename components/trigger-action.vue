<script setup lang="ts">
import { onMounted, onUnmounted, } from 'vue'

declare global {
  interface Window {
    demos: string[];
  }
}

const props = defineProps({
  click: {
    type: String,
    default: 1,
  },
  port: {
    type: Number,
    default: 3710,
  },
  command: {
    type: String,
    required: true,
  },
  args: {
    type: String,
    default: undefined
  },
});

const ws = new WebSocket(`ws://localhost:${props.port}`);

onMounted(() => {
  ws.onopen = function () {
    const isDisabled = document.location.search.includes('disable');
    if (isDisabled) {
      console.log(`Demo disabled: ${props.command}`);
      return;
    }

    const body: { command: string, args?: string[] } = {
      command: props.command,
    };

    if (props.args) {
      body.args = [props.args];
    }

    window.demos = window.demos || [];
    const id = `${props.command}-${JSON.stringify(body.args)}`;
    if (!window.demos.includes(id)) {
      ws.send(JSON.stringify(body));
      window.demos = [...window.demos, id];
    }
  };
});

onUnmounted(() => {
  ws.close();
});
</script>

<template>
  <div class="hidden">{{ props.command }}</div>
</template>
