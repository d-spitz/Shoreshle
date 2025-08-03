<script lang="ts">
	import { fade } from "svelte/transition";

  let { message, show = false, onClose = () => {}, type = 'warning' } = $props();
  let timeout: ReturnType<typeof setTimeout> | null = null;

  $effect(() => {
    if (show) {
      if (timeout) {
        clearTimeout(timeout);
      }
      timeout = setTimeout(() => {
        onClose();
      }, 2200);
    }
  });
</script>

{#if show}
  <div class="alert {type}" transition:fade>{message}</div>
{/if}

<style>
.alert {
  position: fixed;
  top: 2rem;
  left: 50%;
  transform: translateX(-50%);
  color: #222;
  padding: 0.7rem 1.4rem;
  border-radius: 0.7rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.12);
  font-size: 1.1rem;
  z-index: 2000;
  border: 1px solid;
}
.alert.warning {
  background: #ffeb3b;
  border-color: #fbc02d;
}
.alert.success {
  background: #4caf50;
  border-color: #388e3c;
  color: #fff;
}

</style>
