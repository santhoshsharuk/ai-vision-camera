# Camera Interaction App - API Documentation

## Overview

The Camera Interaction App is a web-based application that provides real-time camera feed analysis using AI vision models. It captures video from the user's camera and processes it with natural language instructions using the HuggingFace Transformers library.

## Table of Contents

- [Core Components](#core-components)
- [Public Functions](#public-functions)
- [Configuration Options](#configuration-options)
- [Usage Examples](#usage-examples)
- [Error Handling](#error-handling)
- [Browser Compatibility](#browser-compatibility)

## Core Components

### Video Container (`#videoContainer`)
The main video display container with loading overlay functionality.

**HTML Structure:**
```html
<div id="videoContainer">
  <video id="videoFeed" autoplay playsinline muted></video>
  <div id="loadingOverlay">Loading...</div>
</div>
```

**CSS Classes:**
- `.hidden` - Hides elements from display
- `#loadingOverlay` - Overlay shown during model loading

### Control Panel (`div.controls`)
Interactive controls for managing the camera and processing settings.

**Components:**
- Interval selector dropdown
- Start/Stop button
- Switch Camera button (dynamically added)

### Input/Output Areas (`div.io-areas`)
Text areas for instruction input and response output.

**Components:**
- `#instructionText` - User instruction textarea
- `#responseText` - AI response display textarea (readonly)

## Public Functions

### `initModel()`
Initializes the AI model and processor for vision analysis.

**Description:** Loads the SmolVLM-500M-Instruct model from HuggingFace, with automatic WebGPU/CPU fallback.

**Returns:** `Promise<void>`

**Example:**
```javascript
await initModel();
// Model will be loaded and ready for inference
```

**Side Effects:**
- Sets global `processor` and `model` variables
- Updates `modelLoaded` flag
- Shows/hides loading overlay
- Updates response text with loading status

### `startCamera(facingMode?: string)`
Initializes camera access and starts video stream.

**Parameters:**
- `facingMode` (optional): `"environment"` (back camera) or `"user"` (front camera)

**Returns:** `Promise<void>`

**Example:**
```javascript
// Start back camera (default)
await startCamera();

// Start front camera
facingMode = "user";
await startCamera();
```

**Error Handling:**
- Displays error message in response textarea
- Shows browser alert for camera access failures
- Logs errors to console

### `switchCamera()`
Toggles between front and back camera.

**Description:** Switches the camera facing mode and restarts the camera stream.

**Returns:** `void`

**Example:**
```javascript
switchCamera();
// Camera will switch from back to front or vice versa
```

### `captureImage()`
Captures current video frame as a RawImage object.

**Returns:** `RawImage | null`

**Example:**
```javascript
const image = captureImage();
if (image) {
  // Process the captured image
  console.log(`Captured ${image.width}x${image.height} image`);
}
```

**Returns `null` when:**
- No active video stream
- Video dimensions not available

### `runInference(img, instruction)`
Performs AI inference on the captured image with given instruction.

**Parameters:**
- `img`: `RawImage` - The image to analyze
- `instruction`: `string` - Natural language instruction for analysis

**Returns:** `Promise<string>` - AI-generated response

**Example:**
```javascript
const image = captureImage();
const response = await runInference(image, "What objects do you see?");
console.log(response); // "I can see a laptop, coffee cup, and notebook on the desk."
```

### `sendData()`
Captures image and runs inference with current instruction.

**Description:** Main processing function that combines image capture and AI inference.

**Returns:** `Promise<void>`

**Example:**
```javascript
// Set instruction
document.getElementById('instructionText').value = "Describe the scene";

// Process current video frame
await sendData();
// Response will appear in responseText textarea
```

### `sleep(ms)`
Utility function for creating delays.

**Parameters:**
- `ms`: `number` - Milliseconds to wait

**Returns:** `Promise<void>`

**Example:**
```javascript
await sleep(1000); // Wait 1 second
```

### `processingLoop()`
Main processing loop for continuous analysis.

**Description:** Continuously processes video frames at specified intervals while `isProcessing` is true.

**Returns:** `Promise<void>`

**Example:**
```javascript
// Set processing interval
document.getElementById('intervalSelect').value = "1000";

// Start processing loop
isProcessing = true;
await processingLoop();
```

### `handleStart()`
Starts the processing session.

**Description:** Initiates camera (if needed) and begins continuous processing.

**Returns:** `Promise<void>`

**Side Effects:**
- Sets `isProcessing = true`
- Updates button state and text
- Disables instruction and interval controls
- Starts processing loop

### `handleStop()`
Stops the processing session.

**Description:** Halts continuous processing and re-enables controls.

**Returns:** `void`

**Side Effects:**
- Sets `isProcessing = false`
- Updates button state and text
- Re-enables instruction and interval controls

## Configuration Options

### Interval Settings
Available processing intervals between requests:

| Value | Description |
|-------|-------------|
| 0     | No delay (continuous) |
| 100   | 100 milliseconds |
| 250   | 250 milliseconds |
| 500   | 500 milliseconds |
| 1000  | 1 second |
| 2000  | 2 seconds |

### Model Configuration
- **Model ID:** `HuggingFaceTB/SmolVLM-500M-Instruct`
- **WebGPU Options:** fp16 embeddings, q4 quantization for vision encoder and decoder
- **CPU Fallback:** Automatic fallback when WebGPU unavailable
- **Max New Tokens:** 100 tokens per response

### Camera Settings
- **Default Facing Mode:** `"environment"` (back camera)
- **Alternative Mode:** `"user"` (front camera)
- **Audio:** Disabled
- **Video Constraints:** Automatic resolution based on device capabilities

## Usage Examples

### Basic Setup
```javascript
// The app auto-initializes on page load
window.addEventListener("DOMContentLoaded", async () => {
  await initModel();
  // App is ready for use
});
```

### Custom Instruction Processing
```javascript
// Set custom instruction
document.getElementById('instructionText').value = "Count the number of people in the image";

// Process single frame
await sendData();

// Get response
const response = document.getElementById('responseText').value;
console.log(response);
```

### Continuous Monitoring
```javascript
// Set 2-second interval
document.getElementById('intervalSelect').value = "2000";

// Start continuous processing
await handleStart();

// Stop after 30 seconds
setTimeout(() => {
  handleStop();
}, 30000);
```

### Camera Management
```javascript
// Switch to front camera
switchCamera();

// Wait for camera to initialize
await new Promise(resolve => {
  const checkCamera = () => {
    if (document.getElementById('responseText').value.includes('Camera ready')) {
      resolve();
    } else {
      setTimeout(checkCamera, 100);
    }
  };
  checkCamera();
});
```

## Error Handling

### Common Error Scenarios

1. **Camera Access Denied**
   ```javascript
   // Error appears in responseText
   "Error accessing camera: NotAllowedError: Permission denied"
   ```

2. **Model Loading Failure**
   ```javascript
   // Check modelLoaded flag
   if (!modelLoaded) {
     console.log("Model not loaded yet");
   }
   ```

3. **Inference Errors**
   ```javascript
   // Errors caught and displayed in responseText
   "Error: Failed to process image"
   ```

### Error Recovery
- Camera errors: User must grant permissions and refresh
- Model errors: Page refresh required
- Inference errors: Automatic retry on next cycle

## Browser Compatibility

### Requirements
- **WebRTC Support:** Required for camera access
- **Canvas API:** Required for image capture
- **ES6+ Modules:** Required for HuggingFace Transformers
- **WebGPU:** Optional (falls back to CPU)

### Supported Browsers
- Chrome 88+ (recommended)
- Firefox 85+
- Safari 14+
- Edge 88+

### Mobile Support
- Responsive design with mobile-optimized controls
- Touch-friendly interface
- Automatic camera orientation handling

## Performance Considerations

### WebGPU vs CPU
- **WebGPU:** ~2-5x faster inference, lower battery usage
- **CPU:** Broader compatibility, higher latency

### Memory Usage
- Model size: ~500MB when loaded
- Video processing: Minimal memory footprint
- Continuous processing: Stable memory usage

### Optimization Tips
1. Use longer intervals for battery conservation
2. Stop processing when not needed
3. Prefer back camera for better image quality
4. Ensure good lighting conditions for better AI responses

## Security Considerations

- Camera access requires user permission
- All processing happens locally in browser
- No data sent to external servers
- Model loaded from CDN (HuggingFace)

## Troubleshooting

### Common Issues

1. **"Model not loaded yet"**
   - Wait for initialization to complete
   - Check network connection for model download

2. **"Capture failed"**
   - Ensure camera is active and video is playing
   - Check camera permissions

3. **"WebGPU unavailable"**
   - Normal fallback behavior
   - Performance will be slower but functional

4. **Blank or black video**
   - Check camera permissions
   - Try switching cameras
   - Ensure no other apps are using camera