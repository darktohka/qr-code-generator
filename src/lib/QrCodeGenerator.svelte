<script lang="ts">
  import { onMount } from 'svelte';
  import qrcode from 'qrcode';

  export let text: string = 'Hello!';
  export let errorCorrectionLevel: 'L' | 'M' | 'Q' | 'H' = 'Q';
  export let qrCodeSvg: string = '';
  export let logoDataUrl: string | null = null;
  export let logoScale: number = 25;
  export let logoBorderRadius: number = 0;
  export let logoMargin: number = 0;
  export let pngPixels: number = 300;
  export let colorType: 'solid' | 'gradient' = 'solid';
  export let solidColor: string = '#000000';
  export let gradientType: 'linear' | 'radial' = 'linear';
  export let gradientAngle: number = 0;
  export let gradientColors: { color: string; position: number }[] = [
    { color: '#000000', position: 0 },
    { color: '#ffffff', position: 100 },
  ];

  // Function to generate QR code
  async function generateQrCode() {
    try {
      let qrCodeOptions = {
        errorCorrectionLevel: errorCorrectionLevel,
        type: 'svg' as 'svg', // Explicitly cast to 'svg' literal type
        margin: 1,
        color: {
          dark: colorType === 'solid' ? solidColor : '#ffffff', // Use transparent for gradient, will be replaced by SVG gradient
          light: '#ffffff',
        },
      };

      qrCodeSvg = await qrcode.toString(text, qrCodeOptions);

      if (colorType === 'gradient') {
        qrCodeSvg = applyGradientToSvg(
          qrCodeSvg,
          gradientType,
          gradientAngle,
          gradientColors,
        );
      }
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
        logoMargin,
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
    margin: number,
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

    const logoAreaSize = svgWidth * (scale / 100); // Total area for logo including margin
    const logoAreaX = (svgWidth - logoAreaSize) / 2;
    const logoAreaY = (svgHeight - logoAreaSize) / 2;

    const marginPixels = logoAreaSize * (margin / 100);
    const logoSizeWithMargin = logoAreaSize - 2 * marginPixels;
    const logoXWithMargin = logoAreaX + marginPixels;
    const logoYWithMargin = logoAreaY + marginPixels;

    // Define a unique ID for the clipPath
    const clipPathId = `logoClipPath-${Date.now()}`;

    // Create a clipPath element
    const clipPath = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'clipPath',
    );
    clipPath.setAttribute('id', clipPathId);

    // Calculate rx and ry based on borderRadius percentage
    const rx = (logoAreaSize / 2) * (borderRadius / 50);
    const ry = (logoAreaSize / 2) * (borderRadius / 50);

    // Create a rectangle for the clipPath with rounded corners
    const clipRect = document.createElementNS(
      'http://www.w3.org/2000/svg',
      'rect',
    );
    clipRect.setAttribute('x', logoAreaX.toString());
    clipRect.setAttribute('y', logoAreaY.toString());
    clipRect.setAttribute('width', logoAreaSize.toString());
    clipRect.setAttribute('height', logoAreaSize.toString());
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
    rect.setAttribute('x', logoAreaX.toString());
    rect.setAttribute('y', logoAreaY.toString());
    rect.setAttribute('width', logoAreaSize.toString());
    rect.setAttribute('height', logoAreaSize.toString());
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
    image.setAttribute('x', logoXWithMargin.toString());
    image.setAttribute('y', logoYWithMargin.toString());
    image.setAttribute('width', logoSizeWithMargin.toString());
    image.setAttribute('height', logoSizeWithMargin.toString());
    image.setAttribute('preserveAspectRatio', 'xMidYMid meet');
    image.setAttribute('clip-path', `url(#${clipPathId})`); // Apply the clipPath

    svgElement.appendChild(rect);
    svgElement.appendChild(image);

    return new XMLSerializer().serializeToString(svgElement);
  }

  function applyGradientToSvg(
    svgString: string,
    gradientType: 'linear' | 'radial',
    gradientAngle: number,
    gradientColors: { color: string; position: number }[],
  ): string {
    const parser = new DOMParser();
    const svgDoc = parser.parseFromString(svgString, 'image/svg+xml');
    const svgElement = svgDoc.querySelector('svg');

    if (!svgElement) {
      console.error('SVG element not found in generated QR code for gradient.');
      return svgString;
    }

    let defs = svgElement.querySelector('defs');
    if (!defs) {
      defs = document.createElementNS('http://www.w3.org/2000/svg', 'defs');
      svgElement.prepend(defs);
    }

    const gradientId = `qrGradient-${Date.now()}`;
    let gradientElement: SVGGradientElement;

    if (gradientType === 'linear') {
      gradientElement = document.createElementNS(
        'http://www.w3.org/2000/svg',
        'linearGradient',
      );
      gradientElement.setAttribute('id', gradientId);
      gradientElement.setAttribute('gradientUnits', 'userSpaceOnUse');

      // Calculate x1, y1, x2, y2 based on angle
      // Angle 0deg is left to right. 90deg is top to bottom.
      const angleRad = (gradientAngle - 90) * (Math.PI / 180); // Adjust for SVG's default 0deg (up)
      const r = Math.sqrt(2); // Max distance in a 1x1 square
      const x1 = 0.5 + (r * Math.cos(angleRad)) / 2;
      const y1 = 0.5 + (r * Math.sin(angleRad)) / 2;
      const x2 = 0.5 - (r * Math.cos(angleRad)) / 2;
      const y2 = 0.5 - (r * Math.sin(angleRad)) / 2;

      gradientElement.setAttribute('x1', `${x1 * 100}%`);
      gradientElement.setAttribute('y1', `${y1 * 100}%`);
      gradientElement.setAttribute('x2', `${x2 * 100}%`);
      gradientElement.setAttribute('y2', `${y2 * 100}%`);
    } else {
      // Radial gradient
      gradientElement = document.createElementNS(
        'http://www.w3.org/2000/svg',
        'radialGradient',
      );
      gradientElement.setAttribute('id', gradientId);
      gradientElement.setAttribute('cx', '50%');
      gradientElement.setAttribute('cy', '50%');
      gradientElement.setAttribute('r', '50%');
      gradientElement.setAttribute('fx', '50%');
      gradientElement.setAttribute('fy', '50%');
    }

    gradientColors.forEach((stop) => {
      const stopElement = document.createElementNS(
        'http://www.w3.org/2000/svg',
        'stop',
      );
      stopElement.setAttribute('offset', `${stop.position}%`);
      stopElement.setAttribute('stop-color', stop.color);
      gradientElement.appendChild(stopElement);
    });

    defs.appendChild(gradientElement);

    // Apply gradient to all path elements (QR code modules)
    const paths = svgElement.querySelectorAll('path');
    paths.forEach((path) => {
      path.setAttribute('fill', `url(#${gradientId})`);
    });

    return new XMLSerializer().serializeToString(svgElement);
  }

  // Function to calculate complementary color
  function calculateComplementaryColor(hex: string): string {
    // Remove '#' if present
    const cleanHex = hex.startsWith('#') ? hex.slice(1) : hex;

    // Parse r, g, b values
    const r = parseInt(cleanHex.substring(0, 2), 16);
    const g = parseInt(cleanHex.substring(2, 4), 16);
    const b = parseInt(cleanHex.substring(4, 6), 16);

    // Find the complementary color
    const rComp = (255 - r).toString(16).padStart(2, '0');
    const gComp = (255 - g).toString(16).padStart(2, '0');
    const bComp = (255 - b).toString(16).padStart(2, '0');

    return `#${rComp}${gComp}${bComp}`;
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
  let previousColorType: 'solid' | 'gradient' = colorType;

  $: {
    if (colorType === 'gradient' && previousColorType === 'solid') {
      const complementaryColor = calculateComplementaryColor(solidColor);
      gradientColors = [
        { color: complementaryColor, position: 0 },
        { color: solidColor, position: 100 },
      ];
      gradientAngle = 115;
    }
    previousColorType = colorType;

    text,
      errorCorrectionLevel,
      logoDataUrl,
      logoScale,
      logoBorderRadius,
      logoMargin,
      colorType,
      solidColor,
      gradientType,
      gradientAngle,
      gradientColors,
      generateQrCode();
  }

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
  {logoMargin}
  {pngPixels}
  {colorType}
  {solidColor}
  {gradientType}
  {gradientAngle}
  {gradientColors}
  {handleLogoUpload}
  {exportSvg}
  {exportPng}
/>
