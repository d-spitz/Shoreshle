
<svelte:options runes={true} />
<script lang="ts">
	import { fade } from "svelte/transition";

  let { message, show = false, onClose = () => {} } = $props();
  let timeout: NodeJS.Timeout | null = null;

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
  <div class="alert" transition:fade>{message}</div>
{/if}

<style>
.alert {
  position: fixed;
  top: 2rem;
  left: 50%;
  transform: translateX(-50%);
  background: #ffeb3b;
  color: #222;
  padding: 0.7rem 1.4rem;
  border-radius: 0.7rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.12);
  font-size: 1.1rem;
  z-index: 2000;
  border: 1px solid #fbc02d;
}

</style>
