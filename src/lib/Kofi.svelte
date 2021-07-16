<script lang="ts">
  import { onMount } from 'svelte';

  export let name: string;

  let kofiReady = false;
  let mounted = false;

  onMount(() => {
    mounted = true;
    // if (kofiReady) {
    setTimeout(() => loadKofiWidget(), 1000);
    // }
  });

  // Not working, I dunno why...
  function kofiLoaded() {
    kofiReady = true;
    if (mounted) {
      loadKofiWidget();
    }
  }

  function loadKofiWidget() {
    window.kofiWidgetOverlay?.draw(
      name,
      {
        type: 'floating-chat',
        'floating-chat.donateButton.text': 'สนับสนุนค่า ☕️',
        'floating-chat.donateButton.background-color': '#ff3e00',
        'floating-chat.donateButton.text-color': '#fff',
      },
      'kofiContainer'
    );
  }
</script>

<svelte:head>
  <script
    async
    defer
    type="text/javascript"
    src="https://storage.ko-fi.com/cdn/scripts/overlay-widget.js"
    on:load={kofiLoaded}></script>
</svelte:head>

{#if name}
  <div id="kofiContainer" class="web-only" />
{/if}
