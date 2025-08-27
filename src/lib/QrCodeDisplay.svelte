<script lang="ts">
  export let qrCodeSvg: string;
</script>

<div class="flex-1 flex items-center justify-center p-6 lg:p-0">
  <div
    class="flex justify-center p-6 bg-white border border-gray-200 rounded-lg shadow-inner w-full max-w-xs lg:max-w-none"
  >
    {#if qrCodeSvg}
      {@html qrCodeSvg}
    {:else}
      <p class="text-gray-500 text-center py-8">
        Enter text to generate QR code
      </p>
    {/if}
  </div>
</div>
