<script lang="ts">
  import { onMount } from 'svelte';
  import {
    adler32,
    blake2b,
    blake2s,
    blake3,
    crc32,
    crc64,
    keccak,
    md4,
    md5,
    ripemd160,
    sha1,
    sha224,
    sha256,
    sha3,
    sha384,
    sha512,
    sm3,
    whirlpool,
    xxhash32,
    xxhash64,
    xxhash3,
    xxhash128,
    argon2i,
    argon2d,
    argon2id,
    bcrypt,
    pbkdf2,
    scrypt,
    createHMAC,
    createSHA1,
    createSHA256,
    createSHA512,
  } from 'hash-wasm';

  let password = '';
  let selectedAlgorithm = 'bcrypt';
  let hashedPassword = '';
  let hashedPasswordToVerify = '';
  let error = '';
  let verifyPassword = '';
  let verificationResult: boolean | null = null;

  // Bcrypt configurations
  let bcryptRounds = 10;

  // PBKDF2 configurations
  let pbkdf2Iterations = 100000;
  let pbkdf2KeyLen = 32;
  let pbkdf2HashAlgorithm: 'SHA-1' | 'SHA-256' | 'SHA-512' = 'SHA-256';

  // Argon2 configurations
  let argon2Memory = 65536; // in KiB
  let argon2Iterations = 2;
  let argon2Parallelism = 1;
  let argon2HashLength = 32;

  // Scrypt configurations
  let scryptCostFactor = 1024;
  let scryptBlockSize = 8;
  let scryptParallelism = 1;
  let scryptHashLength = 32;

  // HMAC configurations
  let hmacKey = 'secret';
  let hmacHashFunction: 'SHA-1' | 'SHA-256' | 'SHA-512' = 'SHA-256';

  // BLAKE2b configurations
  let blake2bBits = 512;
  let blake2bKey = '';

  // BLAKE2s configurations
  let blake2sBits = 256;
  let blake2sKey = '';

  // BLAKE3 configurations
  let blake3Bits = 256;
  let blake3Key = '';

  // CRC32 configurations
  let crc32Polynomial = 0xedb88320;

  // CRC64 configurations
  let crc64Polynomial = 'c96c5795d7870f42';

  // Keccak configurations
  let keccakBits: 224 | 256 | 384 | 512 = 512;

  // SHA3 configurations
  let sha3Bits: 224 | 256 | 384 | 512 = 512;

  // xxHash configurations
  let xxhash32Seed = 0;
  let xxhash64SeedLow = 0;
  let xxhash64SeedHigh = 0;
  let xxhash3SeedLow = 0;
  let xxhash3SeedHigh = 0;
  let xxhash128SeedLow = 0;
  let xxhash128SeedHigh = 0;

  const algorithms = [
    { name: 'Adler-32', value: 'adler32' },
    { name: 'Argon2d', value: 'argon2d' },
    { name: 'Argon2i', value: 'argon2i' },
    { name: 'Argon2id', value: 'argon2id' },
    { name: 'Bcrypt', value: 'bcrypt' },
    { name: 'BLAKE2b', value: 'blake2b' },
    { name: 'BLAKE2s', value: 'blake2s' },
    { name: 'BLAKE3', value: 'blake3' },
    { name: 'CRC32', value: 'crc32' },
    { name: 'CRC64', value: 'crc64' },
    { name: 'HMAC (SHA-256)', value: 'hmac_sha256' },
    { name: 'MD4', value: 'md4' },
    { name: 'MD5', value: 'md5' },
    { name: 'PBKDF2', value: 'pbkdf2' },
    { name: 'RIPEMD-160', value: 'ripemd160' },
    { name: 'scrypt', value: 'scrypt' },
    { name: 'SHA-1', value: 'sha1' },
    { name: 'SHA-224', value: 'sha224' },
    { name: 'SHA-256', value: 'sha256' },
    { name: 'SHA-384', value: 'sha384' },
    { name: 'SHA-512', value: 'sha512' },
    { name: 'SHA3-224', value: 'sha3_224' },
    { name: 'SHA3-256', value: 'sha3_256' },
    { name: 'SHA3-384', value: 'sha3_384' },
    { name: 'SHA3-512', value: 'sha3_512' },
    { name: 'Keccak-224', value: 'keccak_224' },
    { name: 'Keccak-256', value: 'keccak_256' },
    { name: 'Keccak-384', value: 'keccak_384' },
    { name: 'Keccak-512', value: 'keccak_512' },
    { name: 'SM3', value: 'sm3' },
    { name: 'Whirlpool', value: 'whirlpool' },
    { name: 'xxHash32', value: 'xxhash32' },
    { name: 'xxHash64', value: 'xxhash64' },
    { name: 'xxHash3', value: 'xxhash3' },
    { name: 'xxHash128', value: 'xxhash128' },
  ];

  async function hashPassword() {
    error = '';
    hashedPassword = '';
    try {
      const passwordBuffer = new TextEncoder().encode(
        password.normalize('NFKC'),
      );
      const salt = crypto.getRandomValues(new Uint8Array(16));

      switch (selectedAlgorithm) {
        case 'adler32':
          hashedPassword = await adler32(passwordBuffer);
          break;
        case 'argon2d':
          hashedPassword = await argon2d({
            password: passwordBuffer,
            salt,
            parallelism: argon2Parallelism,
            iterations: argon2Iterations,
            memorySize: argon2Memory,
            hashLength: argon2HashLength,
            outputType: 'encoded',
          });
          break;
        case 'argon2i':
          hashedPassword = await argon2i({
            password: passwordBuffer,
            salt,
            parallelism: argon2Parallelism,
            iterations: argon2Iterations,
            memorySize: argon2Memory,
            hashLength: argon2HashLength,
            outputType: 'encoded',
          });
          break;
        case 'argon2id':
          hashedPassword = await argon2id({
            password: passwordBuffer,
            salt,
            parallelism: argon2Parallelism,
            iterations: argon2Iterations,
            memorySize: argon2Memory,
            hashLength: argon2HashLength,
            outputType: 'encoded',
          });
          break;
        case 'bcrypt':
          hashedPassword = await bcrypt({
            password: passwordBuffer,
            salt,
            costFactor: bcryptRounds,
            outputType: 'encoded',
          });
          break;
        case 'blake2b':
          hashedPassword = await blake2b(
            passwordBuffer,
            blake2bBits,
            blake2bKey ? new TextEncoder().encode(blake2bKey) : undefined,
          );
          break;
        case 'blake2s':
          hashedPassword = await blake2s(
            passwordBuffer,
            blake2sBits,
            blake2sKey ? new TextEncoder().encode(blake2sKey) : undefined,
          );
          break;
        case 'blake3':
          hashedPassword = await blake3(
            passwordBuffer,
            blake3Bits,
            blake3Key ? new TextEncoder().encode(blake3Key) : undefined,
          );
          break;
        case 'crc32':
          hashedPassword = await crc32(passwordBuffer, crc32Polynomial);
          break;
        case 'crc64':
          hashedPassword = await crc64(passwordBuffer, crc64Polynomial);
          break;
        case 'hmac_sha256':
          const hmacSha256 = await createHMAC(createSHA256(), hmacKey);
          hmacSha256.init();
          hmacSha256.update(passwordBuffer);
          hashedPassword = hmacSha256.digest();
          break;
        case 'md4':
          hashedPassword = await md4(passwordBuffer);
          break;
        case 'md5':
          hashedPassword = await md5(passwordBuffer);
          break;
        case 'pbkdf2':
          let hashFunc;
          if (pbkdf2HashAlgorithm === 'SHA-1') {
            hashFunc = createSHA1();
          } else if (pbkdf2HashAlgorithm === 'SHA-256') {
            hashFunc = createSHA256();
          } else {
            hashFunc = createSHA512();
          }
          hashedPassword = await pbkdf2({
            password: passwordBuffer,
            salt,
            iterations: pbkdf2Iterations,
            hashLength: pbkdf2KeyLen,
            hashFunction: hashFunc,
            outputType: 'hex',
          });
          break;
        case 'ripemd160':
          hashedPassword = await ripemd160(passwordBuffer);
          break;
        case 'scrypt':
          hashedPassword = await scrypt({
            password: passwordBuffer,
            salt,
            costFactor: scryptCostFactor,
            blockSize: scryptBlockSize,
            parallelism: scryptParallelism,
            hashLength: scryptHashLength,
            outputType: 'hex',
          });
          break;
        case 'sha1':
          hashedPassword = await sha1(passwordBuffer);
          break;
        case 'sha224':
          hashedPassword = await sha224(passwordBuffer);
          break;
        case 'sha256':
          hashedPassword = await sha256(passwordBuffer);
          break;
        case 'sha384':
          hashedPassword = await sha384(passwordBuffer);
          break;
        case 'sha512':
          hashedPassword = await sha512(passwordBuffer);
          break;
        case 'sha3_224':
          hashedPassword = await sha3(passwordBuffer, 224);
          break;
        case 'sha3_256':
          hashedPassword = await sha3(passwordBuffer, 256);
          break;
        case 'sha3_384':
          hashedPassword = await sha3(passwordBuffer, 384);
          break;
        case 'sha3_512':
          hashedPassword = await sha3(passwordBuffer, 512);
          break;
        case 'keccak_224':
          hashedPassword = await keccak(passwordBuffer, 224);
          break;
        case 'keccak_256':
          hashedPassword = await keccak(passwordBuffer, 256);
          break;
        case 'keccak_384':
          hashedPassword = await keccak(passwordBuffer, 384);
          break;
        case 'keccak_512':
          hashedPassword = await keccak(passwordBuffer, 512);
          break;
        case 'sm3':
          hashedPassword = await sm3(passwordBuffer);
          break;
        case 'whirlpool':
          hashedPassword = await whirlpool(passwordBuffer);
          break;
        case 'xxhash32':
          hashedPassword = await xxhash32(passwordBuffer, xxhash32Seed);
          break;
        case 'xxhash64':
          hashedPassword = await xxhash64(
            passwordBuffer,
            xxhash64SeedLow,
            xxhash64SeedHigh,
          );
          break;
        case 'xxhash3':
          hashedPassword = await xxhash3(
            passwordBuffer,
            xxhash3SeedLow,
            xxhash3SeedHigh,
          );
          break;
        case 'xxhash128':
          hashedPassword = await xxhash128(
            passwordBuffer,
            xxhash128SeedLow,
            xxhash128SeedHigh,
          );
          break;
        default:
          hashedPassword = 'Algorithm not supported';
      }
    } catch (e: any) {
      console.error('Hashing error:', e);
      error = e.message || 'An unknown error occurred during hashing.';
    }
  }

  // Helper to extract salt from bcrypt hash (simplified)
  function extractBcryptSalt(hash: string): Uint8Array {
    // Bcrypt hash format: $2a$[cost]$[salt][hash]
    // Salt is typically 22 characters long (base64 encoded)
    const parts = hash.split('$');
    if (parts.length >= 3) {
      const saltBase64 = parts[3].substring(0, 22); // Extract the salt part
      // Convert base64 salt to Uint8Array. This is a simplified approach.
      // A proper bcrypt library would handle this.
      try {
        return new Uint8Array(
          atob(saltBase64)
            .split('')
            .map((char) => char.charCodeAt(0)),
        );
      } catch (e) {
        console.warn('Failed to decode bcrypt salt, using dummy salt.', e);
      }
    }
    return crypto.getRandomValues(new Uint8Array(16)); // Fallback to a dummy salt
  }

  async function verifyHashedPassword() {
    error = '';
    verificationResult = null;
    if (!verifyPassword || !hashedPasswordToVerify) {
      return;
    }

    try {
      const passwordBuffer = new TextEncoder().encode(
        verifyPassword.normalize('NFKC'),
      );

      // Autodetect algorithm and verify
      if (
        hashedPasswordToVerify.startsWith('$2a$') ||
        hashedPasswordToVerify.startsWith('$2b$') ||
        hashedPasswordToVerify.startsWith('$2y$')
      ) {
        // Bcrypt
        const costFactor = parseInt(hashedPasswordToVerify.split('$')[2], 10);
        const salt = extractBcryptSalt(hashedPasswordToVerify);

        const rehashed = await bcrypt({
          password: passwordBuffer,
          salt,
          costFactor,
          outputType: 'encoded',
        });
        verificationResult = rehashed === hashedPasswordToVerify;
      } else if (hashedPasswordToVerify.startsWith('$argon2')) {
        // Argon2
        const argon2Parts = hashedPasswordToVerify.split('$');
        if (argon2Parts.length >= 6) {
          const type = argon2Parts[1]
            .substring(0, argon2Parts[1].indexOf('v='))
            .replace('argon2', '');
          const version = parseInt(argon2Parts[1].split('v=')[1], 10);
          const params = argon2Parts[2].split(',');
          let mem = 65536;
          let iter = 2;
          let par = 1;
          params.forEach((p) => {
            if (p.startsWith('m=')) mem = parseInt(p.substring(2), 10);
            if (p.startsWith('t=')) iter = parseInt(p.substring(2), 10);
            if (p.startsWith('p=')) par = parseInt(p.substring(2), 10);
          });
          const saltBase64 = argon2Parts[4];
          const hashBase64 = argon2Parts[5];

          const saltBuffer = new Uint8Array(
            atob(saltBase64)
              .split('')
              .map((char) => char.charCodeAt(0)),
          );
          const hashLength = 32;

          let rehashed;
          if (type === 'i') {
            rehashed = await argon2i({
              password: passwordBuffer,
              salt: saltBuffer,
              parallelism: par,
              iterations: iter,
              memorySize: mem,
              hashLength: hashLength,
              outputType: 'encoded',
            });
          } else if (type === 'd') {
            rehashed = await argon2d({
              password: passwordBuffer,
              salt: saltBuffer,
              parallelism: par,
              iterations: iter,
              memorySize: mem,
              hashLength: hashLength,
              outputType: 'encoded',
            });
          } else {
            // argon2id
            rehashed = await argon2id({
              password: passwordBuffer,
              salt: saltBuffer,
              parallelism: par,
              iterations: iter,
              memorySize: mem,
              hashLength: hashLength,
              outputType: 'encoded',
            });
          }
          verificationResult = rehashed === hashedPasswordToVerify;
        } else {
          error = 'Invalid Argon2 hash format.';
          verificationResult = false;
        }
      } else if (
        hashedPasswordToVerify.length === 40 &&
        /^[0-9a-fA-F]+$/.test(hashedPasswordToVerify)
      ) {
        // SHA-1 (hex encoded, 40 chars)
        const rehashed = await sha1(passwordBuffer);
        verificationResult = rehashed === hashedPasswordToVerify;
      } else if (
        hashedPasswordToVerify.length === 64 &&
        /^[0-9a-fA-F]+$/.test(hashedPasswordToVerify)
      ) {
        // SHA-256 (hex encoded, 64 chars)
        const rehashed = await sha256(passwordBuffer);
        verificationResult = rehashed === hashedPasswordToVerify;
      } else if (
        hashedPasswordToVerify.length === 128 &&
        /^[0-9a-fA-F]+$/.test(hashedPasswordToVerify)
      ) {
        // SHA-512 (hex encoded, 128 chars)
        const rehashed = await sha512(passwordBuffer);
        verificationResult = rehashed === hashedPasswordToVerify;
      } else {
        error = 'Could not autodetect hash algorithm for verification.';
        verificationResult = false;
      }
    } catch (e: any) {
      console.error('Verification error:', e);
      error = e.message || 'An unknown error occurred during verification.';
      verificationResult = false;
    }
  }

  // Reactive statement to re-hash when inputs change
  $: password,
    selectedAlgorithm,
    bcryptRounds,
    pbkdf2Iterations,
    pbkdf2KeyLen,
    pbkdf2HashAlgorithm,
    argon2Memory,
    argon2Iterations,
    argon2Parallelism,
    argon2HashLength,
    scryptCostFactor,
    scryptBlockSize,
    scryptParallelism,
    scryptHashLength,
    hmacKey,
    hmacHashFunction,
    blake2bBits,
    blake2bKey,
    blake2sBits,
    blake2sKey,
    blake3Bits,
    blake3Key,
    crc32Polynomial,
    crc64Polynomial,
    keccakBits,
    sha3Bits,
    xxhash32Seed,
    xxhash64SeedLow,
    xxhash64SeedHigh,
    xxhash3SeedLow,
    xxhash3SeedHigh,
    xxhash128SeedLow,
    xxhash128SeedHigh,
    hashPassword();

  // Reactive statement for verification
  $: verifyPassword, hashedPasswordToVerify, verifyHashedPassword();

  onMount(() => {
    hashPassword();
  });

  async function copyToClipboard() {
    try {
      await navigator.clipboard.writeText(hashedPassword);
      alert('Hashed password copied to clipboard!');
    } catch (err) {
      console.error('Failed to copy: ', err);
    }
  }
</script>

<div class="bg-white shadow-2xl rounded-xl p-8 w-full max-w-md mx-auto">
  <h2 class="text-2xl font-bold mb-6 text-center">Password Hasher</h2>

  <div class="space-y-4">
    <div>
      <label for="password" class="block text-sm font-medium text-gray-700"
        >PASSWORD TO HASH</label
      >
      <input
        type="text"
        id="password"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={password}
      />
    </div>

    <div>
      <label
        for="selectedAlgorithm"
        class="block text-sm font-medium text-gray-700">ALGORITHM</label
      >
      <select
        id="selectedAlgorithm"
        class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
        bind:value={selectedAlgorithm}
      >
        {#each algorithms as algo}
          <option value={algo.value}>{algo.name}</option>
        {/each}
      </select>
    </div>

    {#if selectedAlgorithm === 'bcrypt'}
      <div>
        <label
          for="bcryptRounds"
          class="block text-sm font-medium text-gray-700">BCRYPT ROUNDS</label
        >
        <input
          type="number"
          id="bcryptRounds"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={bcryptRounds}
          min="4"
          max="31"
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'argon2d' || selectedAlgorithm === 'argon2i' || selectedAlgorithm === 'argon2id'}
      <div>
        <label
          for="argon2Memory"
          class="block text-sm font-medium text-gray-700"
          >ARGON2 MEMORY (KiB)</label
        >
        <input
          type="number"
          id="argon2Memory"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={argon2Memory}
          min="8"
        />
      </div>
      <div>
        <label
          for="argon2Iterations"
          class="block text-sm font-medium text-gray-700"
          >ARGON2 ITERATIONS</label
        >
        <input
          type="number"
          id="argon2Iterations"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={argon2Iterations}
          min="1"
        />
      </div>
      <div>
        <label
          for="argon2Parallelism"
          class="block text-sm font-medium text-gray-700"
          >ARGON2 PARALLELISM</label
        >
        <input
          type="number"
          id="argon2Parallelism"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={argon2Parallelism}
          min="1"
        />
      </div>
      <div>
        <label
          for="argon2HashLength"
          class="block text-sm font-medium text-gray-700"
          >ARGON2 HASH LENGTH</label
        >
        <input
          type="number"
          id="argon2HashLength"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={argon2HashLength}
          min="16"
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'pbkdf2'}
      <div>
        <label
          for="pbkdf2Iterations"
          class="block text-sm font-medium text-gray-700"
          >PBKDF2 ITERATIONS</label
        >
        <input
          type="number"
          id="pbkdf2Iterations"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={pbkdf2Iterations}
          min="1000"
        />
      </div>
      <div>
        <label
          for="pbkdf2KeyLen"
          class="block text-sm font-medium text-gray-700"
          >PBKDF2 KEY LENGTH (bytes)</label
        >
        <input
          type="number"
          id="pbkdf2KeyLen"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={pbkdf2KeyLen}
          min="16"
        />
      </div>
      <div>
        <label
          for="pbkdf2HashAlgorithm"
          class="block text-sm font-medium text-gray-700"
          >PBKDF2 HASH ALGORITHM</label
        >
        <select
          id="pbkdf2HashAlgorithm"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={pbkdf2HashAlgorithm}
        >
          <option value="SHA-1">SHA-1</option>
          <option value="SHA-256">SHA-256</option>
          <option value="SHA-512">SHA-512</option>
        </select>
      </div>
    {/if}

    {#if selectedAlgorithm === 'scrypt'}
      <div>
        <label
          for="scryptCostFactor"
          class="block text-sm font-medium text-gray-700"
          >SCRYPT COST FACTOR (N)</label
        >
        <input
          type="number"
          id="scryptCostFactor"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={scryptCostFactor}
          min="2"
        />
      </div>
      <div>
        <label
          for="scryptBlockSize"
          class="block text-sm font-medium text-gray-700"
          >SCRYPT BLOCK SIZE (r)</label
        >
        <input
          type="number"
          id="scryptBlockSize"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={scryptBlockSize}
          min="1"
        />
      </div>
      <div>
        <label
          for="scryptParallelism"
          class="block text-sm font-medium text-gray-700"
          >SCRYPT PARALLELISM (p)</label
        >
        <input
          type="number"
          id="scryptParallelism"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={scryptParallelism}
          min="1"
        />
      </div>
      <div>
        <label
          for="scryptHashLength"
          class="block text-sm font-medium text-gray-700"
          >SCRYPT HASH LENGTH (bytes)</label
        >
        <input
          type="number"
          id="scryptHashLength"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={scryptHashLength}
          min="16"
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'hmac_sha256'}
      <div>
        <label for="hmacKey" class="block text-sm font-medium text-gray-700"
          >HMAC KEY</label
        >
        <input
          type="text"
          id="hmacKey"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={hmacKey}
        />
      </div>
      <div>
        <label
          for="hmacHashFunction"
          class="block text-sm font-medium text-gray-700"
          >HMAC HASH FUNCTION</label
        >
        <select
          id="hmacHashFunction"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={hmacHashFunction}
        >
          <option value="SHA-1">SHA-1</option>
          <option value="SHA-256">SHA-256</option>
          <option value="SHA-512">SHA-512</option>
        </select>
      </div>
    {/if}

    {#if selectedAlgorithm === 'blake2b'}
      <div>
        <label for="blake2bBits" class="block text-sm font-medium text-gray-700"
          >BLAKE2b BITS</label
        >
        <input
          type="number"
          id="blake2bBits"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake2bBits}
          min="8"
          max="512"
        />
      </div>
      <div>
        <label for="blake2bKey" class="block text-sm font-medium text-gray-700"
          >BLAKE2b KEY</label
        >
        <input
          type="text"
          id="blake2bKey"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake2bKey}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'blake2s'}
      <div>
        <label for="blake2sBits" class="block text-sm font-medium text-gray-700"
          >BLAKE2s BITS</label
        >
        <input
          type="number"
          id="blake2sBits"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake2sBits}
          min="8"
          max="256"
        />
      </div>
      <div>
        <label for="blake2sKey" class="block text-sm font-medium text-gray-700"
          >BLAKE2s KEY</label
        >
        <input
          type="text"
          id="blake2sKey"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake2sKey}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'blake3'}
      <div>
        <label for="blake3Bits" class="block text-sm font-medium text-gray-700"
          >BLAKE3 BITS</label
        >
        <input
          type="number"
          id="blake3Bits"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake3Bits}
          min="8"
          max="512"
        />
      </div>
      <div>
        <label for="blake3Key" class="block text-sm font-medium text-gray-700"
          >BLAKE3 KEY</label
        >
        <input
          type="text"
          id="blake3Key"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={blake3Key}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'crc32'}
      <div>
        <label
          for="crc32Polynomial"
          class="block text-sm font-medium text-gray-700"
          >CRC32 POLYNOMIAL</label
        >
        <input
          type="number"
          id="crc32Polynomial"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={crc32Polynomial}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'crc64'}
      <div>
        <label
          for="crc64Polynomial"
          class="block text-sm font-medium text-gray-700"
          >CRC64 POLYNOMIAL</label
        >
        <input
          type="text"
          id="crc64Polynomial"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={crc64Polynomial}
        />
      </div>
    {/if}

    {#if selectedAlgorithm.startsWith('keccak_')}
      <div>
        <label for="keccakBits" class="block text-sm font-medium text-gray-700"
          >KECCAK BITS</label
        >
        <select
          id="keccakBits"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={keccakBits}
        >
          <option value={224}>224</option>
          <option value={256}>256</option>
          <option value={384}>384</option>
          <option value={512}>512</option>
        </select>
      </div>
    {/if}

    {#if selectedAlgorithm.startsWith('sha3_')}
      <div>
        <label for="sha3Bits" class="block text-sm font-medium text-gray-700"
          >SHA3 BITS</label
        >
        <select
          id="sha3Bits"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={sha3Bits}
        >
          <option value={224}>224</option>
          <option value={256}>256</option>
          <option value={384}>384</option>
          <option value={512}>512</option>
        </select>
      </div>
    {/if}

    {#if selectedAlgorithm === 'xxhash32'}
      <div>
        <label
          for="xxhash32Seed"
          class="block text-sm font-medium text-gray-700">XXHASH32 SEED</label
        >
        <input
          type="number"
          id="xxhash32Seed"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash32Seed}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'xxhash64'}
      <div>
        <label
          for="xxhash64SeedLow"
          class="block text-sm font-medium text-gray-700"
          >XXHASH64 SEED LOW</label
        >
        <input
          type="number"
          id="xxhash64SeedLow"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash64SeedLow}
        />
      </div>
      <div>
        <label
          for="xxhash64SeedHigh"
          class="block text-sm font-medium text-gray-700"
          >XXHASH64 SEED HIGH</label
        >
        <input
          type="number"
          id="xxhash64SeedHigh"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash64SeedHigh}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'xxhash3'}
      <div>
        <label
          for="xxhash3SeedLow"
          class="block text-sm font-medium text-gray-700"
          >XXHASH3 SEED LOW</label
        >
        <input
          type="number"
          id="xxhash3SeedLow"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash3SeedLow}
        />
      </div>
      <div>
        <label
          for="xxhash3SeedHigh"
          class="block text-sm font-medium text-gray-700"
          >XXHASH3 SEED HIGH</label
        >
        <input
          type="number"
          id="xxhash3SeedHigh"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash3SeedHigh}
        />
      </div>
    {/if}

    {#if selectedAlgorithm === 'xxhash128'}
      <div>
        <label
          for="xxhash128SeedLow"
          class="block text-sm font-medium text-gray-700"
          >XXHASH128 SEED LOW</label
        >
        <input
          type="number"
          id="xxhash128SeedLow"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash128SeedLow}
        />
      </div>
      <div>
        <label
          for="xxhash128SeedHigh"
          class="block text-sm font-medium text-gray-700"
          >XXHASH128 SEED HIGH</label
        >
        <input
          type="number"
          id="xxhash128SeedHigh"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={xxhash128SeedHigh}
        />
      </div>
    {/if}

    {#if error}
      <div class="text-red-500 text-center">{error}</div>
    {/if}

    <div
      class="relative bg-gray-100 p-4 rounded-md text-center text-xl font-bold break-all flex items-center justify-center"
    >
      {hashedPassword || 'Enter password and select algorithm'}
      {#if hashedPassword}
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
      {/if}
    </div>

    <div class="mt-6 pt-6 border-t border-gray-200">
      <h3 class="text-xl font-bold mb-4 text-center">Verify Password</h3>
      <div>
        <label
          for="verifyPassword"
          class="block text-sm font-medium text-gray-700"
          >PASSWORD TO VERIFY</label
        >
        <input
          type="text"
          id="verifyPassword"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={verifyPassword}
        />
      </div>
      <div class="mt-4">
        <label
          for="hashedPasswordToVerify"
          class="block text-sm font-medium text-gray-700"
          >HASH TO VERIFY AGAINST</label
        >
        <input
          type="text"
          id="hashedPasswordToVerify"
          class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2"
          bind:value={hashedPasswordToVerify}
        />
      </div>
      {#if verificationResult !== null}
        <div class="mt-4 text-center text-lg font-semibold">
          {#if verificationResult}
            <span class="text-green-600">Verification Successful!</span>
          {:else}
            <span class="text-red-600">Verification Failed.</span>
          {/if}
        </div>
      {/if}
    </div>
  </div>
</div>
