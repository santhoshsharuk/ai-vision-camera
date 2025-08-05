# Quick Reference Guide

## Common Use Cases

### 1. Basic Image Analysis
```javascript
// Set instruction
document.getElementById('instructionText').value = "What objects are in this image?";

// Process single frame
await sendData();

// Get result
const result = document.getElementById('responseText').value;
```

### 2. Continuous Monitoring
```javascript
// Set 2-second interval
document.getElementById('intervalSelect').value = "2000";

// Start continuous processing
await handleStart();

// Stop processing
handleStop();
```

### 3. Switch Camera
```javascript
// Toggle between front/back camera
switchCamera();
```

### 4. Custom Instructions
```javascript
const instructions = [
  "Count the number of people",
  "Describe the colors you see",
  "What is the person doing?",
  "Is this a safe environment?",
  "What text can you read?"
];

// Use any instruction
document.getElementById('instructionText').value = instructions[0];
```

## Code Snippets

### Check Model Status
```javascript
if (modelLoaded) {
  console.log("Model is ready");
} else {
  console.log("Model still loading...");
}
```

### Check Camera Status
```javascript
if (stream && stream.active) {
  console.log("Camera is active");
} else {
  console.log("Camera not available");
}
```

### Manual Image Capture
```javascript
const image = captureImage();
if (image) {
  const response = await runInference(image, "Describe this image");
  console.log(response);
}
```

### Get Processing Interval
```javascript
const interval = parseInt(document.getElementById('intervalSelect').value, 10);
console.log(`Current interval: ${interval}ms`);
```

### Check Processing Status
```javascript
if (isProcessing) {
  console.log("Currently processing");
} else {
  console.log("Processing stopped");
}
```

## Event Listeners

### Listen for Camera Events
```javascript
// Custom event listeners (if implemented)
window.addEventListener('camera:started', (e) => {
  console.log('Camera started:', e.detail);
});

window.addEventListener('camera:error', (e) => {
  console.log('Camera error:', e.detail);
});
```

### Button State Changes
```javascript
// Monitor start button changes
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    if (mutation.attributeName === 'class') {
      const isStop = startButton.classList.contains('stop');
      console.log(`Button state: ${isStop ? 'Stop' : 'Start'}`);
    }
  });
});

observer.observe(startButton, { attributes: true });
```

## Troubleshooting

### Common Issues & Solutions

#### "Model not loaded yet"
```javascript
// Wait for model to load
while (!modelLoaded) {
  await sleep(1000);
}
console.log("Model is now ready");
```

#### Camera Permission Denied
```javascript
// Check permissions
navigator.permissions.query({name: 'camera'}).then((result) => {
  console.log('Camera permission:', result.state);
  if (result.state === 'denied') {
    alert('Please enable camera permissions and refresh the page');
  }
});
```

#### WebGPU Not Available
```javascript
if (!navigator.gpu) {
  console.log("WebGPU not available, using CPU");
  // Performance will be slower but functional
}
```

#### Processing Too Slow
```javascript
// Increase interval to reduce load
document.getElementById('intervalSelect').value = "2000"; // 2 seconds

// Or stop processing temporarily
if (isProcessing) {
  handleStop();
  console.log("Processing stopped to reduce load");
}
```

### Debug Information

#### Get System Info
```javascript
const systemInfo = {
  userAgent: navigator.userAgent,
  webGPU: !!navigator.gpu,
  mediaDevices: !!navigator.mediaDevices,
  getUserMedia: !!navigator.mediaDevices?.getUserMedia,
  modelLoaded: modelLoaded,
  cameraActive: stream?.active || false,
  processing: isProcessing
};

console.log("System Info:", systemInfo);
```

#### Performance Metrics
```javascript
// Measure inference time
const startTime = performance.now();
await sendData();
const endTime = performance.now();
console.log(`Processing took ${endTime - startTime}ms`);
```

#### Memory Usage (Chrome only)
```javascript
if (performance.memory) {
  console.log('Memory usage:', {
    used: Math.round(performance.memory.usedJSHeapSize / 1024 / 1024) + 'MB',
    total: Math.round(performance.memory.totalJSHeapSize / 1024 / 1024) + 'MB',
    limit: Math.round(performance.memory.jsHeapSizeLimit / 1024 / 1024) + 'MB'
  });
}
```

## Customization Examples

### Change Model Settings
```javascript
// Before model loading, modify options
const customOptions = {
  dtype: {
    embed_tokens: "fp32",  // Higher precision
    vision_encoder: "fp16",
    decoder_model_merged: "fp16"
  },
  device: "webgpu"
};
```

### Custom Preprocessing
```javascript
function customCaptureImage() {
  const image = captureImage();
  if (!image) return null;
  
  // Apply custom preprocessing
  return preprocessImage(image, {
    brightness: 1.2,
    contrast: 1.1
  });
}
```

### Batch Processing
```javascript
async function processBatch() {
  const instructions = [
    "What do you see?",
    "Count objects",
    "Describe colors"
  ];
  
  const results = [];
  for (const instruction of instructions) {
    document.getElementById('instructionText').value = instruction;
    await sendData();
    await sleep(1000); // Wait between requests
    results.push(document.getElementById('responseText').value);
  }
  
  return results;
}
```

## CSS Customization

### Change Button Colors
```css
#startButton.start {
  background-color: #007bff; /* Blue instead of green */
}

#startButton.stop {
  background-color: #fd7e14; /* Orange instead of red */
}

.switch-camera {
  background-color: #6f42c1; /* Purple */
}
```

### Custom Loading Overlay
```css
#loadingOverlay {
  background: linear-gradient(45deg, rgba(0,0,0,0.8), rgba(0,0,0,0.6));
  font-size: 1.2em;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 0.8; }
  50% { opacity: 1; }
}
```

### Responsive Adjustments
```css
@media (max-width: 480px) {
  #videoContainer {
    max-width: 100%;
    margin: 0 -16px; /* Full width on small screens */
  }
  
  .controls {
    flex-direction: column;
    gap: 8px;
  }
}
```

## Integration Patterns

### With Existing Forms
```javascript
// Add to existing form
const form = document.getElementById('myForm');
const hiddenInput = document.createElement('input');
hiddenInput.type = 'hidden';
hiddenInput.name = 'aiResponse';

form.appendChild(hiddenInput);

// Update hidden input with AI response
function updateFormWithResponse() {
  hiddenInput.value = document.getElementById('responseText').value;
}
```

### With Local Storage
```javascript
// Save responses to local storage
function saveResponse(instruction, response) {
  const responses = JSON.parse(localStorage.getItem('aiResponses') || '[]');
  responses.push({
    timestamp: new Date().toISOString(),
    instruction,
    response
  });
  localStorage.setItem('aiResponses', JSON.stringify(responses));
}

// Load previous responses
function loadResponses() {
  return JSON.parse(localStorage.getItem('aiResponses') || '[]');
}
```

### With Analytics
```javascript
// Track usage
function trackAnalysis(instruction, responseTime) {
  // Google Analytics example
  if (typeof gtag !== 'undefined') {
    gtag('event', 'ai_analysis', {
      'custom_parameter_1': instruction.length,
      'custom_parameter_2': responseTime
    });
  }
}
```

## Keyboard Shortcuts

### Add Keyboard Controls
```javascript
document.addEventListener('keydown', (e) => {
  // Space bar to start/stop
  if (e.code === 'Space' && !e.target.matches('textarea')) {
    e.preventDefault();
    startButton.click();
  }
  
  // 'C' to switch camera
  if (e.code === 'KeyC' && !e.target.matches('textarea')) {
    switchCamera();
  }
  
  // 'Enter' in instruction field to process
  if (e.code === 'Enter' && e.target.id === 'instructionText') {
    if (!isProcessing) {
      e.preventDefault();
      sendData();
    }
  }
});
```

## Mobile Optimizations

### Touch Gestures
```javascript
// Add touch support for camera switching
let touchStartX = 0;

videoContainer.addEventListener('touchstart', (e) => {
  touchStartX = e.touches[0].clientX;
});

videoContainer.addEventListener('touchend', (e) => {
  const touchEndX = e.changedTouches[0].clientX;
  const diff = touchStartX - touchEndX;
  
  // Swipe left/right to switch camera
  if (Math.abs(diff) > 50) {
    switchCamera();
  }
});
```

### Prevent Zoom on Double Tap
```css
#videoContainer {
  touch-action: manipulation; /* Prevents zoom on double tap */
}
```

This quick reference provides practical code snippets and solutions for common scenarios when working with the Camera Interaction App.