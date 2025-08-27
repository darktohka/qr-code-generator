<script lang="ts">
  export let pngPixels: number;
  export let onExportSvg: () => void;
  export let onExportPng: () => void;
  export let onPngPixelsChange: (pixels: number) => void;
</script>

<div class="flex flex-col space-y-4 mt-auto pt-6 border-t border-gray-200">
  <button
    class="bg-gradient-to-r from-blue-500 to-blue-600 hover:from-blue-600 hover:to-blue-700 text-white font-bold py-2.5 px-5 rounded-lg shadow-md hover:shadow-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-all duration-200"
    on:click={onExportSvg}
  >
    Export as SVG
  </button>
  <div class="flex items-center space-x-4">
    <input
      type="number"
      value={pngPixels}
      on:input={(e) =>
        onPngPixelsChange(parseInt((e.target as HTMLInputElement).value))}
      min="50"
      max="1200"
      step="50"
      class="shadow-sm border border-gray-300 rounded-lg py-2.5 px-4 text-gray-800 leading-tight focus:outline-none focus:ring-2 focus:ring-green-500 focus:border-transparent transition-all duration-200 w-28"
    />
    <span>px</span>
    <button
      class="flex-grow bg-gradient-to-r from-green-500 to-green-600 hover:from-green-600 hover:to-green-700 text-white font-bold py-2.5 px-5 rounded-lg shadow-md hover:shadow-lg focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50 transition-all duration-200"
      on:click={onExportPng}
    >
      Export as PNG
    </button>
  </div>
</div>
