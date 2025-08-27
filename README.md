# QR Code Generator

This is a modern and customizable QR Code Generator application built with Svelte and TypeScript.

It allows users to generate QR codes with various customization options, including text/URL encoding, error correction levels, custom colors (solid or gradient), embedded logos, and export QR codes to SVG and PNG formats.

## Features

- **Live Demo**: Check out the live application at [https://qr.tohka.us](https://qr.tohka.us).
- **Text/URL Encoding**: Encode any text or URL into a QR code.
- **Error Correction Levels**: Choose from different error correction levels (L, M, Q, H) to control the QR code's resilience to damage.
- **Customizable Colors**:
  - **Solid Color**: Set a single solid color for your QR code.
  - **Gradient Colors**: Apply linear or radial gradients with multiple color stops and adjustable angles for linear gradients.
- **Embedded Logo**:
  - Upload a custom image to embed as a logo in the center of the QR code.
  - Adjust the logo's size, border-radius, and margin for perfect placement.
- **Export Options**:
  - **SVG Export**: Download the generated QR code as a scalable vector graphic (SVG).
  - **PNG Export**: Download the QR code as a Portable Network Graphics (PNG) image with adjustable pixel dimensions.

## Technologies Used

- [Svelte](https://svelte.dev/): A cybernetically enhanced web framework.
- [TypeScript](https://www.typescriptlang.org/): A strongly typed superset of JavaScript that compiles to plain JavaScript.
- [Tailwind CSS](https://tailwindcss.com/): A utility-first CSS framework for rapidly building custom designs.
- [Vite](https://vitejs.dev/): A next-generation frontend tooling that provides an extremely fast development experience.
- [qrcode.js](https://github.com/davidshimjs/qrcodejs): A JavaScript library for generating QR codes.

## Getting Started

### Prerequisites

Make sure you have [Bun](https://bun.sh) installed.

### Installation

1.  Clone the repository:
    ```bash
    git clone https://github.com/darktohka/qr-code-generator.git
    cd qr-code-generator
    ```
2.  Install dependencies:
    ```bash
    bun install
    ```

### Running the Development Server

```bash
bun run dev
```

Open [http://localhost:5173](http://localhost:5173) (or the port shown in your terminal) in your browser to see the application.

### Building for Production

```bash
bun run build
```

This will create a `dist` directory with the production-ready build.
