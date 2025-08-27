<script lang="ts">
  import { onMount } from 'svelte';
  import qrcode from 'qrcode';

  export let text: string = 'Hello!';
  export let errorCorrectionLevel: 'L' | 'M' | 'Q' | 'H' = 'Q';
  export let qrCodeSvg: string = '';
  export let logoDataUrl: string | null = null;
  export let logoScale: number = 25;
  export let logoBorderRadius: number = 0;
  export let pngPixels: number = 300;

  // Function to generate QR code
  async function generateQrCode() {
    try {
      qrCodeSvg = await qrcode.toString(text, {
        errorCorrectionLevel: errorCorrectionLevel,
        type: 'svg',
        margin: 1,
        color: {
          dark: '#000000',
          light: '#ffffff',
        },
      });
    } catch (err) {
      console.error(err);
      qrCodeSvg = '<p class="text-red-500">Error generating QR code.</p>';
    }

    if (logoDataUrl && qrCodeSvg) {
      qrCodeSvg = addLogoToSvg(
        qrCodeSvg,
        logoDataUrl,
        logoScale,
        logoBorderRadius,
      );
    }
  }

  // Function to handle logo upload
  export function handleLogoUpload(event: Event) {
    const input = event.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      const reader = new FileReader();
      reader.onload = (e) => {
        logoDataUrl = e.target?.result as string;
      };
      reader.readAsDataURL(input.files[0]);
    } else {
      logoDataUrl = null;
    }
  }

  // Function to export as SVG
  export function exportSvg() {
    const blob = new Blob([qrCodeSvg], { type: 'image/svg+xml' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'qrcode.svg';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function addLogoToSvg(
    svgString: string,
    logoUrl: string,
    scale: number,
    borderRadius: number,
  ): string {
    const parser = new DOMParser();
    const svgDoc = parser.parseFromString(svgString, 'image/svg+xml');
    const svgElement = svgDoc.querySelector('svg');

    if (!svgElement) {
      console.error('SVG element not found in generated QR code.');
      return svgString;
    }

    const viewBox = svgElement.getAttribute('viewBox');
    let svgWidth: number;
    let svgHeight: number;

    if (viewBox) {
      const parts = viewBox.split(' ');
      svgWidth = parseFloat(parts[2]);
      svgHeight = parseFloat(parts[3]);
    } else {
      svgWidth = parseFloat(svgElement.getAttribute('width') || '256');
      svgHeight = parseFloat(svgElement.getAttribute('height') || '256');
    }

    const logoSize = svgWidth * (scale / 100);
    const logoX = (svgWidth - logoSize) / 2;
    const logoY = (svgHeight - logoSize) / 2;

    // Define a unique ID for the clipPath
    const clipPathId = `logoClipPath-${Date.now()}`;

    // Create a clipPath element
    const clipPath = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'clipPath',
    );
    clipPath.setAttribute('id', clipPathId);

    // Calculate rx and ry based on borderRadius percentage
    const rx = (logoSize / 2) * (borderRadius / 50);
    const ry = (logoSize / 2) * (borderRadius / 50);

    // Create a rectangle for the clipPath with rounded corners
    const clipRect = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'rect',
    );
    clipRect.setAttribute('x', logoX.toString());
    clipRect.setAttribute('y', logoY.toString());
    clipRect.setAttribute('width', logoSize.toString());
    clipRect.setAttribute('height', logoSize.toString());
    clipRect.setAttribute('rx', rx.toString());
    clipRect.setAttribute('ry', ry.toString());

    clipPath.appendChild(clipRect);

    // Append the clipPath to the defs section (or directly to svg if defs not present)
    let defs = svgElement.querySelector('defs');
    if (!defs) {
      defs = document.createElementNS('http://www.w3.org/2000/svg', 'defs');
      svgElement.prepend(defs); // Prepend to ensure it's available before use
    }
    defs.appendChild(clipPath);

    // Create a white rectangle to cut out the QR code area for the logo
    const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
    rect.setAttribute('x', logoX.toString());
    rect.setAttribute('y', logoY.toString());
    rect.setAttribute('width', logoSize.toString());
    rect.setAttribute('height', logoSize.toString());
    rect.setAttribute('fill', '#ffffff'); // White background
    // Apply the same border radius to the background rectangle
    rect.setAttribute('rx', rx.toString());
    rect.setAttribute('ry', ry.toString());

    // Create the image element for the logo
    const image = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'image',
    );
    image.setAttribute('href', logoUrl);
    image.setAttribute('x', logoX.toString());
    image.setAttribute('y', logoY.toString());
    image.setAttribute('width', logoSize.toString());
    image.setAttribute('height', logoSize.toString());
    image.setAttribute('preserveAspectRatio', 'xMidYMid meet');
    image.setAttribute('clip-path', `url(#${clipPathId})`); // Apply the clipPath

    svgElement.appendChild(rect);
    svgElement.appendChild(image);

    return new XMLSerializer().serializeToString(svgElement);
  }

  // Function to export as PNG
  export async function exportPng() {
    try {
      if (!qrCodeSvg) {
        console.error('No QR code SVG to export.');
        return;
      }

      const svgBlob = new Blob([qrCodeSvg], {
        type: 'image/svg+xml;charset=utf-8',
      });
      const svgUrl = URL.createObjectURL(svgBlob);

      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement('canvas');

        // Get actual dimensions from the SVG to ensure correct scaling
        const parser = new DOMParser();
        const svgDoc = parser.parseFromString(qrCodeSvg, 'image/svg+xml');
        const svgElement = svgDoc.querySelector('svg');
        let svgWidth = parseFloat(svgElement?.getAttribute('width') || '256');
        let svgHeight = parseFloat(svgElement?.getAttribute('height') || '256');
        let aspectRatio = svgWidth / svgHeight;

        canvas.width = pngPixels;
        canvas.height = pngPixels * aspectRatio;

        const ctx = canvas.getContext('2d');
        if (ctx) {
          ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        }

        const url = canvas.toDataURL('image/png');
        const a = document.createElement('a');
        a.href = url;
        a.download = `qrcode_${pngPixels}px.png`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(svgUrl);
      };
      img.onerror = (err) => {
        console.error('Error loading SVG image for PNG export:', err);
        URL.revokeObjectURL(svgUrl);
      };
      img.src = svgUrl;
    } catch (err) {
      console.error(err);
    }
  }

  // Generate QR code on initial load and when text, errorCorrectionLevel, logoDataUrl, logoScale, or logoBorderRadius changes
  $: text,
    errorCorrectionLevel,
    logoDataUrl,
    logoScale,
    logoBorderRadius,
    generateQrCode();

  onMount(() => {
    generateQrCode();
  });
</script>

<slot
  {text}
  {errorCorrectionLevel}
  {qrCodeSvg}
  {logoDataUrl}
  {logoScale}
  {logoBorderRadius}
  {pngPixels}
  {handleLogoUpload}
  {exportSvg}
  {exportPng}
/>
