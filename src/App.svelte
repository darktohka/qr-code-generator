<script lang="ts">
  import QrCodeGenerator from './lib/QrCodeGenerator.svelte';
  import QrCodeInputForm from './lib/QrCodeInputForm.svelte';
  import QrCodeDisplay from './lib/QrCodeDisplay.svelte';
  import QrCodeExportButtons from './lib/QrCodeExportButtons.svelte';
  import TotpGenerator from './lib/TotpGenerator.svelte';

  let currentComponent: 'qr' | 'totp' = 'qr'; // State to manage which component is active

  let text: string;
  let errorCorrectionLevel: 'L' | 'M' | 'Q' | 'H';
  let qrCodeSvg: string;
  let logoDataUrl: string | null;
  let logoScale: number;
  let logoBorderRadius: number;
  let logoMargin: number = 0;
  let pngPixels: number;
  let colorType: 'solid' | 'gradient' = 'solid';
  let solidColor: string = '#000000';
  let gradientType: 'linear' | 'radial' = 'linear';
  let gradientAngle: number = 0;
  let gradientColors: { color: string; position: number }[] = [
    { color: '#000000', position: 0 },
    { color: '#ffffff', position: 100 },
  ];
</script>

<div
  class="min-h-screen flex flex-col items-center justify-center p-6 bg-gray-100"
>
  <div class="mb-6 flex space-x-4">
    <button
      class="px-6 py-3 rounded-lg shadow-md transition-all duration-200 ease-in-out
             {currentComponent === 'qr'
        ? 'bg-blue-600 text-white'
        : 'bg-white text-gray-800 hover:bg-gray-200'}"
      on:click={() => (currentComponent = 'qr')}
    >
      QR Code Generator
    </button>
    <button
      class="px-6 py-3 rounded-lg shadow-md transition-all duration-200 ease-in-out
             {currentComponent === 'totp'
        ? 'bg-blue-600 text-white'
        : 'bg-white text-gray-800 hover:bg-gray-200'}"
      on:click={() => (currentComponent = 'totp')}
    >
      TOTP Generator
    </button>
  </div>

  {#if currentComponent === 'qr'}
    <QrCodeGenerator
      bind:text
      bind:errorCorrectionLevel
      bind:qrCodeSvg
      bind:logoDataUrl
      bind:logoScale
      bind:logoBorderRadius
      bind:logoMargin
      bind:pngPixels
      bind:colorType
      bind:solidColor
      bind:gradientType
      bind:gradientAngle
      bind:gradientColors
      let:exportSvg
      let:exportPng
    >
      <main
        class="bg-white shadow-2xl rounded-xl p-8 w-full w-full max-w-4xl flex flex-col lg:flex-row lg:space-x-6"
      >
        <!-- Left Column: Controls and Export Buttons -->
        <div class="flex-1 flex flex-col space-y-6">
          <QrCodeInputForm
            bind:text
            bind:errorCorrectionLevel
            bind:logoDataUrl
            bind:logoScale
            bind:logoBorderRadius
            bind:logoMargin
            bind:colorType
            bind:solidColor
            bind:gradientType
            bind:gradientAngle
            bind:gradientColors
            onTextChange={(value) => (text = value)}
            onErrorCorrectionChange={(value) => (errorCorrectionLevel = value)}
            onLogoUpload={(value) => (logoDataUrl = value)}
            onRemoveLogo={() => (logoDataUrl = null)}
            onLogoScaleChange={(value) => (logoScale = value)}
            onLogoBorderRadiusChange={(value) => (logoBorderRadius = value)}
            onLogoMarginChange={(value) => (logoMargin = value)}
            onColorTypeChange={(value) => (colorType = value)}
            onSolidColorChange={(value) => (solidColor = value)}
            onGradientTypeChange={(value) => (gradientType = value)}
            onGradientAngleChange={(value) => (gradientAngle = value)}
            onGradientColorsChange={(value) => (gradientColors = value)}
          />

          <QrCodeExportButtons
            bind:pngPixels
            onExportSvg={exportSvg}
            onExportPng={exportPng}
            onPngPixelsChange={(value) => (pngPixels = value)}
          />
        </div>

        <!-- Right Column: QR Code Display -->
        <QrCodeDisplay {qrCodeSvg} />
      </main>
    </QrCodeGenerator>
  {:else if currentComponent === 'totp'}
    <TotpGenerator />
  {/if}
</div>
