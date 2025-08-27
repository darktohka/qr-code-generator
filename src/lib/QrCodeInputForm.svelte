<script lang="ts">
  export let text: string;
  export let errorCorrectionLevel: 'L' | 'M' | 'Q' | 'H';
  export let logoDataUrl: string | null;
  export let logoScale: number;
  export let logoBorderRadius: number;
  export let logoMargin: number;

  export let onTextChange: (text: string) => void;
  export let onErrorCorrectionChange: (level: 'L' | 'M' | 'Q' | 'H') => void;
  export let onLogoUpload: (dataUrl: string | null) => void;
  export let onRemoveLogo: () => void;
  export let onLogoScaleChange: (scale: number) => void;
  export let onLogoBorderRadiusChange: (radius: number) => void;
  export let onLogoMarginChange: (margin: number) => void;
  export let colorType: 'solid' | 'gradient';
  export let solidColor: string;
  export let gradientType: 'linear' | 'radial';
  export let gradientAngle: number;
  export let gradientColors: { color: string; position: number }[];

  export let onColorTypeChange: (type: 'solid' | 'gradient') => void;
  export let onSolidColorChange: (color: string) => void;
  export let onGradientTypeChange: (type: 'linear' | 'radial') => void;
  export let onGradientAngleChange: (angle: number) => void;
  export let onGradientColorsChange: (
    colors: { color: string; position: number }[],
  ) => void;

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

  <fieldset class="mb-7">
    <legend class="block text-gray-800 text-sm font-semibold mb-2">
      QR Code Color Type:
    </legend>
    <div class="flex space-x-4">
      <label class="inline-flex items-center">
        <input
          type="radio"
          class="form-radio text-blue-600"
          name="colorType"
          value="solid"
          checked={colorType === 'solid'}
          on:change={() => onColorTypeChange('solid')}
        />
        <span class="ml-2 text-gray-800">Solid Color</span>
      </label>
      <label class="inline-flex items-center">
        <input
          type="radio"
          class="form-radio text-blue-600"
          name="colorType"
          value="gradient"
          checked={colorType === 'gradient'}
          on:change={() => onColorTypeChange('gradient')}
        />
        <span class="ml-2 text-gray-800">Gradient</span>
      </label>
    </div>
  </fieldset>

  {#if colorType === 'solid'}
    <div class="mb-7">
      <label
        for="solid-color"
        class="block text-gray-800 text-sm font-semibold mb-2"
      >
        Solid Color:
      </label>
      <input
        type="color"
        id="solid-color"
        class="w-full h-10 p-1 border border-gray-300 rounded-lg cursor-pointer"
        value={solidColor}
        on:input={(e) =>
          onSolidColorChange((e.target as HTMLInputElement).value)}
      />
    </div>
  {/if}

  {#if colorType === 'gradient'}
    <fieldset class="mb-7">
      <legend class="block text-gray-800 text-sm font-semibold mb-2">
        Gradient Type:
      </legend>
      <div class="flex space-x-4">
        <label class="inline-flex items-center">
          <input
            type="radio"
            class="form-radio text-blue-600"
            name="gradientType"
            value="linear"
            checked={gradientType === 'linear'}
            on:change={() => onGradientTypeChange('linear')}
          />
          <span class="ml-2 text-gray-800">Linear</span>
        </label>
        <label class="inline-flex items-center">
          <input
            type="radio"
            class="form-radio text-blue-600"
            name="gradientType"
            value="radial"
            checked={gradientType === 'radial'}
            on:change={() => onGradientTypeChange('radial')}
          />
          <span class="ml-2 text-gray-800">Radial</span>
        </label>
      </div>
    </fieldset>

    {#if gradientType === 'linear'}
      <div class="mb-7">
        <label
          for="gradient-angle"
          class="block text-gray-800 text-sm font-semibold mb-2"
        >
          Gradient Angle: {gradientAngle}°
        </label>
        <input
          type="range"
          id="gradient-angle"
          min="0"
          max="360"
          step="1"
          value={gradientAngle}
          on:input={(e) =>
            onGradientAngleChange(
              parseInt((e.target as HTMLInputElement).value),
            )}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700"
        />
      </div>
    {/if}

    <fieldset class="mb-7">
      <legend class="block text-gray-800 text-sm font-semibold mb-2">
        Gradient Colors:
      </legend>
      {#each gradientColors as colorStop, index (index)}
        <div class="flex items-center space-x-2 mb-2">
          <label for="gradient-color-{index}" class="sr-only"
            >Color {index + 1}</label
          >
          <input
            type="color"
            id="gradient-color-{index}"
            class="w-10 h-10 p-1 border border-gray-300 rounded-lg cursor-pointer"
            value={colorStop.color}
            on:input={(e) => {
              gradientColors[index].color = (
                e.target as HTMLInputElement
              ).value;
              onGradientColorsChange(gradientColors);
            }}
          />
          <label for="gradient-position-{index}" class="sr-only"
            >Position {index + 1}</label
          >
          <input
            type="range"
            id="gradient-position-{index}"
            min="0"
            max="100"
            step="1"
            value={colorStop.position}
            on:input={(e) => {
              gradientColors[index].position = parseInt(
                (e.target as HTMLInputElement).value,
              );
              onGradientColorsChange(gradientColors);
            }}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700"
          />
          <span class="text-gray-800">{colorStop.position}%</span>
          <button
            on:click={() => {
              gradientColors.splice(index, 1);
              onGradientColorsChange(gradientColors);
            }}
            class="bg-red-500 hover:bg-red-600 text-white font-bold py-1 px-2 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-all duration-200 text-xs"
          >
            Remove
          </button>
        </div>
      {/each}
      <button
        on:click={() => {
          gradientColors = [
            ...gradientColors,
            { color: '#ffffff', position: 100 },
          ];
          onGradientColorsChange(gradientColors);
        }}
        class="mt-2 bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-all duration-200"
      >
        Add Color Stop
      </button>
    </fieldset>
  {/if}

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

      <div class="mb-7">
        <label
          for="logo-margin"
          class="block text-gray-800 text-sm font-semibold mb-2"
        >
          Logo Margin: {logoMargin}%
        </label>
        <input
          type="range"
          id="logo-margin"
          min="0"
          max="25"
          step="1"
          value={logoMargin}
          on:input={(e) =>
            onLogoMarginChange(parseInt((e.target as HTMLInputElement).value))}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer range-lg dark:bg-gray-700"
        />
      </div>
    {/if}
  </div>
</div>
