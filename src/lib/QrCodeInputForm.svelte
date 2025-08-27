<script lang="ts">
  export let text: string;
  export let errorCorrectionLevel: 'L' | 'M' | 'Q' | 'H';
  export let logoDataUrl: string | null;
  export let logoScale: number;
  export let logoBorderRadius: number;

  export let onTextChange: (text: string) => void;
  export let onErrorCorrectionChange: (level: 'L' | 'M' | 'Q' | 'H') => void;
  export let onLogoUpload: (dataUrl: string | null) => void;
  export let onRemoveLogo: () => void;
  export let onLogoScaleChange: (scale: number) => void;
  export let onLogoBorderRadiusChange: (radius: number) => void;

  function handleLogoFileInput(event: Event) {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const reader = new FileReader();
      reader.onload = (e) => {
        onLogoUpload(e.target?.result as string);
      };
      reader.readAsDataURL(input.files[0]);
    } else {
      onLogoUpload(null);
    }
  }
</script>

<div class="flex-1 flex flex-col space-y-6">
  <h1
    class="text-3xl font-extrabold text-center text-gray-900 mb-6 tracking-tight"
  >
    QR Code Generator
  </h1>

  <div class="mb-5">
    <label for="qr-text" class="block text-gray-800 text-sm font-semibold mb-2">
      Text to encode:
    </label>
    <input
      type="text"
      id="qr-text"
      class="shadow-sm border border-gray-300 rounded-lg w-full py-2.5 px-4 text-gray-800 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
      value={text}
      on:input={(e) => onTextChange((e.target as HTMLInputElement).value)}
      placeholder="Enter text or URL"
    />
  </div>

  <div class="mb-7">
    <label
      for="error-correction"
      class="block text-gray-800 text-sm font-semibold mb-2"
    >
      Error Correction Level:
    </label>
    <select
      id="error-correction"
      class="shadow-sm border border-gray-300 rounded-lg w-full py-2.5 px-4 text-gray-800 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
      value={errorCorrectionLevel}
      on:change={(e) =>
        onErrorCorrectionChange(
          (e.target as HTMLSelectElement).value as 'L' | 'M' | 'Q' | 'H',
        )}
    >
      <option value="L">L (Low)</option>
      <option value="M">M (Medium)</option>
      <option value="Q">Q (Quartile)</option>
      <option value="H">H (High)</option>
    </select>
  </div>

  <div class="mb-7">
    <label
      for="logo-upload"
      class="block text-gray-800 text-sm font-semibold mb-2"
    >
      Upload Logo (Optional):
    </label>
    <input
      type="file"
      id="logo-upload"
      accept="image/*"
      class="shadow-sm border border-gray-300 rounded-lg w-full py-2.5 px-4 text-gray-800 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
      on:change={handleLogoFileInput}
    />
    {#if logoDataUrl}
      <div class="mt-4 flex items-center space-x-4">
        <img
          src={logoDataUrl}
          alt="Uploaded Logo"
          class="h-16 w-16 object-contain border border-gray-300 rounded-lg p-1"
        />
        <button
          on:click={onRemoveLogo}
          class="bg-red-500 hover:bg-red-600 text-white font-bold py-1 px-2 rounded-lg shadow-md focus:outline-none
        focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-all duration-200 text-xs"
        >
          Remove Logo
        </button>
      </div>

      <div class="mb-7">
        <label
          for="logo-scale"
          class="block text-gray-800 text-sm font-semibold mb-2"
        >
          Logo Size: {logoScale}%
        </label>
        <input
          type="range"
          id="logo-scale"
          min="5"
          max="50"
          step="1"
          value={logoScale}
          on:input={(e) =>
            onLogoScaleChange(parseInt((e.target as HTMLInputElement).value))}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700"
        />
      </div>

      <div class="mb-7">
        <label
          for="logo-border-radius"
          class="block text-gray-800 text-sm font-semibold mb-2"
        >
          Logo Border Radius: {logoBorderRadius}%
        </label>
        <input
          type="range"
          id="logo-border-radius"
          min="0"
          max="50"
          step="1"
          value={logoBorderRadius}
          on:input={(e) =>
            onLogoBorderRadiusChange(
              parseInt((e.target as HTMLInputElement).value),
            )}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700"
        />
      </div>
    {/if}
  </div>
</div>
