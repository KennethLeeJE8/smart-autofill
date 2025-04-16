# Smart Autofill Extension Development

This project provides a development environment for testing the Smart Autofill extension. It includes multiple pages that allow testing form autofill capabilities across different sites using the Shared Storage API.

## Important Note

**The Shared Storage API is currently an experimental technology** that is part of the Privacy Sandbox initiative. It's only available in Chrome with the Privacy Sandbox features enabled. The application includes fallbacks to localStorage for browsers that don't support this API.

## Features

- **Page 1 (localhost:8080)**: Main form page for entering personal information, which saves to Shared Storage
- **Page 2 (localhost:8081)**: Read-only page that displays data from Shared Storage
- **Storage Host (localhost:8088)**: Hosts the Shared Storage for cross-site data persistence

The project uses the new Privacy Sandbox Shared Storage API to enable cross-site data sharing while maintaining privacy. If Shared Storage is not available, the application falls back to localStorage.

## How It Works

1. When a user submits data on Page 1 (localhost:8080), it communicates with localhost:8088 via iframe and postMessage to save the data in localhost:8088's Shared Storage
2. Page 2 (localhost:8081) communicates with localhost:8088 in the same way to read the data
3. The Storage Host (localhost:8088) manages its own Shared Storage and handles read/write requests from the other pages
4. All pages include fallbacks to localStorage if the Shared Storage API is not available

### Cross-Origin Communication Flow

1. **Writing Data**: 
   - Page 1 creates a hidden iframe to localhost:8088
   - Once loaded, it posts a message with the data to save
   - localhost:8088 saves the data to its Shared Storage and sends a confirmation back

2. **Reading Data**:
   - Page 2 creates a hidden iframe to localhost:8088
   - It posts a message requesting data from a specific key
   - localhost:8088 uses a worklet to read from its Shared Storage
   - The data is sent back via postMessage to Page 2

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- npm (v6 or higher)
- Chrome browser with Privacy Sandbox features enabled
  - Enable Privacy Sandbox in Chrome: chrome://settings/privacySandbox

### Installation

1. Clone the repository
2. Install dependencies:

```bash
npm install
```

### Running the development servers

Run all three pages simultaneously:

```bash
npm run dev:all
```

Or run individual pages:

```bash
# Run the main input page (localhost:8080)
npm run dev:page1

# Run the read-only page (localhost:8081)
npm run dev:page2

# Run the shared storage host page (localhost:8088)
npm run dev:dummy
```

### Building for production

```bash
npm run build
```

## Project Structure

```
smart-autofill/
├── src/
│   ├── common/            # Shared utilities and styles
│   │   ├── formUtils.ts   # Form handling utilities
│   │   ├── sharedStorageUtils.ts # Shared Storage utilities
│   │   └── styles.css     # Common styles
│   ├── page1/             # Input form page (localhost:8080)
│   │   ├── index.html
│   │   └── index.ts
│   ├── page2/             # Read-only page (localhost:8081)
│   │   ├── index.html
│   │   └── index.ts
│   ├── dummy/             # Shared Storage host (localhost:8088)
│   │   ├── index.html
│   │   └── index.ts
│   ├── shared-storage.d.ts # TypeScript declarations for Shared Storage API
│   ├── sharedStorageReadWorklet.js # Worklet script for reading from Shared Storage
│   └── ...
├── package.json
├── tsconfig.json
├── webpack.config.js
└── README.md
```

## Shared Storage API

This project demonstrates the use of the Privacy Sandbox Shared Storage API, which allows limited cross-site storage of data with privacy protections. Key features used:

1. Writing data to Shared Storage: `window.sharedStorage.set()`
2. Reading data via Worklets: `window.sharedStorage.worklet.addModule()` and `window.sharedStorage.run()`
3. Cross-origin communication via iframes and postMessage
4. Automatic fallback to localStorage when Shared Storage is not available

## Troubleshooting

If you encounter TypeScript errors related to the Shared Storage API, remember that this is an experimental API and TypeScript doesn't have built-in types for it. The project includes type definitions in `src/shared-storage.d.ts`, but you may need to use type assertions (`as any`) when accessing the API.

## License

This project is licensed under the MIT License.