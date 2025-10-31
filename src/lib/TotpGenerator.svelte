<script lang="ts">
  import { TOTP } from 'otpauth';
  import { onMount, onDestroy } from 'svelte';

  let secretKey: string = 'JBSWY3DPEHPK3PXP';
  let digits: number = 6;
  let period: number = 30;
  let algorithm = 'SHA1';
  let totpCode: string = '';
  let timeLeft: number = 0;
  let interval: NodeJS.Timeout;

  function generateTotp() {
    try {
      const totp = new TOTP({
        secret: secretKey,
        digits: digits,
        period: period,
        algorithm: algorithm,
      });
      totpCode = totp.generate();
      timeLeft = totp.remaining() / 1000;
    } catch (error) {
      console.error('Error generating TOTP:', error);
      totpCode = 'Error';
    }
  }

  function startTimer() {
    interval = setInterval(() => {
      timeLeft--;
      if (timeLeft <= 0) {
        generateTotp();
      }
    }, 1000);
  }

  onMount(() => {
    generateTotp();
    startTimer();
  });

  onDestroy(() => {
    clearInterval(interval);
  });

  $: secretKey, digits, period, algorithm, generateTotp();

  async function copyToClipboard() {
    try {
      await navigator.clipboard.writeText(totpCode);
      alert('TOTP code copied to clipboard!');
    } catch (err) {
      console.error('Failed to copy: ', err);
    }
  }
</script>

<div class="bg-white shadow-2xl rounded-xl p-8 w-full max-w-md mx-auto">
  <h2 class="text-2xl font-bold mb-6 text-center">TOTP Token Generator</h2>

  <div class="space-y-4">
    <div>
      <label for="secretKey" class="block text-sm font-medium text-gray-700"
        >YOUR SECRET KEY</label
      >
      <input
        type="text"
        id="secretKey"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={secretKey}
      />
    </div>

    <div>
      <label for="digits" class="block text-sm font-medium text-gray-700"
        >NUMBER OF DIGITS</label
      >
      <input
        type="number"
        id="digits"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={digits}
        min="6"
        max="8"
      />
    </div>

    <div>
      <label for="period" class="block text-sm font-medium text-gray-700"
        >TOKEN PERIOD (IN SECONDS)</label
      >
      <input
        type="number"
        id="period"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={period}
        min="1"
      />
    </div>

    <div>
      <label for="algorithm" class="block text-sm font-medium text-gray-700"
        >HASHING TYPE</label
      >
      <select
        id="algorithm"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={algorithm}
      >
        <option value="SHA1">SHA1</option>
        <option value="SHA224">SHA224</option>
        <option value="SHA256">SHA256</option>
        <option value="SHA384">SHA384</option>
        <option value="SHA512">SHA512</option>
        <option value="SHA3-224">SHA3-224</option>
        <option value="SHA3-256">SHA3-256</option>
        <option value="SHA3-384">SHA3-384</option>
        <option value="SHA3-512">SHA3-512</option>
      </select>
    </div>

    <div class="text-center text-sm text-gray-500">
      Updating in {(timeLeft + 1).toFixed(0)} second{(timeLeft + 1).toFixed(
        0,
      ) === '1'
        ? ''
        : 's'}
    </div>
    <div class="w-full bg-gray-200 rounded-full h-2.5 mb-4">
      <div
        class="bg-blue-600 h-2.5 rounded-full transition-all duration-1000 ease-linear"
        style="width: {100 - ((period - timeLeft) / period) * 100}%"
      ></div>
    </div>

    <div
      class="relative bg-gray-100 p-4 rounded-md text-center text-5xl font-bold tracking-widest flex items-center justify-center"
    >
      {totpCode}
      <button
        on:click={copyToClipboard}
        class="absolute right-4 bottom-4 p-2 bg-gray-200 rounded-md hover:bg-gray-300 hover:cursor-pointer"
        aria-label="Copy to clipboard"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="h-6 w-6 text-gray-600"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
          stroke-width="2"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"
          />
        </svg>
      </button>
    </div>
  </div>
</div>
