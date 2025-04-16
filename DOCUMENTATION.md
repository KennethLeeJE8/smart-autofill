# Shared Storage API Implementation Guide

## Overview

The Shared Storage API is a privacy-preserving mechanism for cross-site storage in web applications. This documentation explains how the API works and provides implementation examples based on our demo project.

## Table of Contents

- [Key Concepts](#key-concepts)
- [Basic Usage](#basic-usage)
- [Use Cases](#use-cases)
- [Implementation Guide](#implementation-guide)
- [Best Practices](#best-practices)

## Key Concepts

### Worklets
Special JavaScript environments that provide controlled access to shared storage data. Worklets are the only context where shared storage values can be read.

### Gates
Controlled mechanisms that determine how information can be extracted from worklets, ensuring privacy preservation.

### Cross-site Storage
The ability to write data on one site and read it on another, while maintaining privacy constraints.

## Basic Usage

### Writing to Shared Storage

```javascript
// Basic write operation
window.sharedStorage.set('key-name', 'value');

// Write with ignore option
window.sharedStorage.set('key-name', 'value', {
    ignoreIfPresent: true
});

// Delete from storage
window.sharedStorage.delete('key-name');
```

### Reading from Shared Storage

```javascript
class SelectURLOperation {
  async run(urls, data) {
    // Read from shared storage within the worklet
    const storedValue = await sharedStorage.get('key-name');
    
    // Process the value and return an index
    return someIndex;
  }
}
```

## Use Cases

### 1. A/B Testing

Implement cross-site A/B testing by storing user group assignments:

```javascript
async function setupABTest() {
  const abTestingWorklet = await window.sharedStorage.createWorklet(
    'ab-testing-worklet.js'
  );

  // Set initial experiment group
  window.sharedStorage.set('ab-testing-group', getRandomExperiment(), {
    ignoreIfPresent: true,
  });

  // Select URL based on experiment group
  const selectedUrl = await abTestingWorklet.selectURL('ab-testing', urls, {
    data: groups,
    resolveToConfig: true
  });
}
```

### 2. Frequency Capping

Control how often users see specific content:

```javascript
class FrequencyCapOperation {
  async run(urls, data) {
    const count = parseInt(await sharedStorage.get('frequency-count'));
    
    if (count === FREQUENCY_LIMIT) {
      return 0; // Show default content
    }
    
    await sharedStorage.set('frequency-count', count + 1);
    return 1; // Show targeted content
  }
}
```

### 3. Creative Rotation

Implement different strategies for rotating content:

```javascript
class CreativeRotationOperation {
  async run(urls, data) {
    const rotationMode = await sharedStorage.get('creative-rotation-mode');
    
    switch (rotationMode) {
      case 'sequential':
        // Sequential rotation logic
        return this.handleSequentialRotation(urls);
      case 'even-distribution':
        // Random rotation with equal probability
        return Math.floor(Math.random() * urls.length);
      case 'weighted-distribution':
        // Weighted probability rotation
        return this.handleWeightedRotation(data);
    }
  }
}
```

## Implementation Guide

### 1. Setup and Feature Detection

```javascript
if ('sharedStorage' in window) {
  // Shared Storage implementation
} else {
  // Fallback implementation
}
```

### 2. Creating a Worklet

```javascript
const worklet = await window.sharedStorage.createWorklet('your-worklet.js');
```

### 3. Defining Worklet Operations

```javascript
// your-worklet.js
class CustomOperation {
  async run(urls, data) {
    const storedValue = await sharedStorage.get('your-key');
    // Implementation logic
    return result;
  }
}
```

### 4. Running Operations

```javascript
const result = await worklet.run('operation-name', {
  data: yourData
});
```

## Best Practices

### Security and Privacy

1. Store only necessary data
2. Use worklets for reading data
3. Avoid storing sensitive information
4. Never attempt to extract private information

### Performance

1. Minimize stored data size
2. Use `ignoreIfPresent` option to prevent unnecessary writes
3. Implement proper cleanup of stored data

### Error Handling

1. Always check for API availability
2. Provide fallback mechanisms
3. Handle storage quotas and limitations
4. Implement proper error reporting

### Cross-Origin Considerations

1. Ensure proper CORS headers
2. Handle cross-origin worklet loading
3. Consider origin-specific storage strategies

## Browser Support and Requirements

To use Shared Storage API:

1. Enable Privacy Sandbox experiment flag: `chrome://flags/#privacy-sandbox-enrollment-overrides`
2. Use Chrome version that supports the API
3. Ensure proper HTTPS implementation for production

## Additional Resources

- [Chrome Privacy Sandbox Documentation](https://developer.chrome.com/docs/privacy-sandbox/shared-storage/)
- [Shared Storage API Specification](https://wicg.github.io/shared-storage/)
- [Privacy Considerations](https://developer.chrome.com/docs/privacy-sandbox/) 