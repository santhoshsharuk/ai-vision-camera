# Component Reference Guide

## Overview

This document provides detailed information about all UI components, their properties, styling, and interactions within the Camera Interaction App.

## HTML Components

### Video Components

#### `#videoFeed`
**Type:** `<video>` element
**Purpose:** Displays live camera stream

**Attributes:**
- `autoplay` - Automatically starts playback
- `playsinline` - Prevents fullscreen on iOS
- `muted` - Prevents audio playback

**Properties:**
- `srcObject` - MediaStream from camera
- `videoWidth` - Native video width (readonly)
- `videoHeight` - Native video height (readonly)

**Events:**
- `loadedmetadata` - Fired when video dimensions are available

**Example Usage:**
```javascript
const video = document.getElementById('videoFeed');
video.srcObject = stream;
console.log(`Video dimensions: ${video.videoWidth}x${video.videoHeight}`);
```

#### `#videoContainer`
**Type:** `<div>` container
**Purpose:** Wrapper for video and loading overlay

**CSS Properties:**
- `position: relative` - Enables absolute positioning for overlay
- `aspect-ratio: 4/3` - Maintains consistent aspect ratio
- `max-width: 480px` - Responsive width constraint

**Child Elements:**
- `#videoFeed` - Video element
- `#loadingOverlay` - Loading indicator

#### `#loadingOverlay`
**Type:** `<div>` overlay
**Purpose:** Shows loading status during model initialization

**CSS Properties:**
- `position: absolute` - Overlays the video
- `display: none` (default) - Hidden by default
- `background-color: rgba(0, 0, 0, 0.7)` - Semi-transparent background
- `z-index: 10` - Above video layer

**States:**
- Hidden: `display: none`
- Visible: `display: flex`

**Example Usage:**
```javascript
const overlay = document.getElementById('loadingOverlay');
overlay.style.display = 'flex'; // Show loading
overlay.style.display = 'none'; // Hide loading
```

### Canvas Element

#### `#canvas`
**Type:** `<canvas>` element
**Purpose:** Off-screen image capture and processing

**CSS Classes:**
- `.hidden` - Visually hidden from user

**Properties:**
- `width` - Set dynamically to match video width
- `height` - Set dynamically to match video height

**Context:**
- 2D rendering context with `willReadFrequently: true` option

**Example Usage:**
```javascript
const canvas = document.getElementById('canvas');
canvas.width = video.videoWidth;
canvas.height = video.videoHeight;
const ctx = canvas.getContext('2d', { willReadFrequently: true });
ctx.drawImage(video, 0, 0);
```

### Input/Output Components

#### `#instructionText`
**Type:** `<textarea>` element
**Purpose:** User input for AI instructions

**Attributes:**
- `name="Instruction"`
- Default value: "What do you see?"

**Properties:**
- `value` - Current instruction text
- `disabled` - Disabled during processing

**CSS Properties:**
- `resize: vertical` - Allows vertical resizing only
- `min-height: 50px` - Minimum height constraint

**States:**
- Enabled: User can edit
- Disabled: Read-only during processing

**Example Usage:**
```javascript
const instruction = document.getElementById('instructionText');
instruction.value = "Describe the colors in the image";
instruction.disabled = true; // Disable during processing
```

#### `#responseText`
**Type:** `<textarea>` element
**Purpose:** Display AI responses and status messages

**Attributes:**
- `name="Response"`
- `readonly` - User cannot edit
- `placeholder="Server response will appear here..."`

**Properties:**
- `value` - Current response or status text

**Usage Patterns:**
- Status updates: "Loading processor…"
- Error messages: "Error accessing camera: ..."
- AI responses: "I can see a laptop and coffee cup..."

**Example Usage:**
```javascript
const response = document.getElementById('responseText');
response.value = "Processing…";
response.value = await runInference(image, instruction);
```

### Control Components

#### `#intervalSelect`
**Type:** `<select>` dropdown
**Purpose:** Configure processing interval

**Options:**
```html
<option value="0" selected>0 ms</option>
<option value="100">100 ms</option>
<option value="250">250 ms</option>
<option value="500">500 ms</option>
<option value="1000">1 s</option>
<option value="2000">2 s</option>
```

**Properties:**
- `value` - Selected interval in milliseconds
- `disabled` - Disabled during processing

**Example Usage:**
```javascript
const intervalSelect = document.getElementById('intervalSelect');
const interval = parseInt(intervalSelect.value, 10);
intervalSelect.disabled = true; // Disable during processing
```

#### `#startButton`
**Type:** `<button>` element
**Purpose:** Start/stop processing control

**CSS Classes:**
- `.start` - Green background (ready to start)
- `.stop` - Red background (ready to stop)

**Properties:**
- `textContent` - "Start" or "Stop"
- `disabled` - Disabled until camera is ready

**States:**
1. **Disabled**: Camera not ready
2. **Start**: Ready to begin processing
3. **Stop**: Currently processing

**Example Usage:**
```javascript
const startButton = document.getElementById('startButton');

// Enable start mode
startButton.textContent = "Start";
startButton.classList.replace("stop", "start");
startButton.disabled = false;

// Enable stop mode
startButton.textContent = "Stop";
startButton.classList.replace("start", "stop");
```

#### Switch Camera Button
**Type:** `<button>` element (dynamically created)
**Purpose:** Toggle between front and back camera

**CSS Classes:**
- `.switch-camera` - Blue background styling

**Creation:**
```javascript
const btn = document.createElement("button");
btn.textContent = "Switch Camera";
btn.classList.add("switch-camera");
btn.addEventListener("click", switchCamera);
document.querySelector(".controls").appendChild(btn);
```

## Container Components

### `.controls`
**Type:** `<div>` container
**Purpose:** Groups all control elements

**Layout:**
- Flexbox with gap spacing
- Responsive: Column layout on mobile

**Child Elements:**
- Interval selector wrapper
- Start/Stop button
- Switch Camera button (added dynamically)

### `.io-areas`
**Type:** `<div>` container
**Purpose:** Groups input and output text areas

**Layout:**
- Flexbox column layout
- Full width with consistent spacing

**Child Elements:**
- Instruction text area with label
- Response text area with label

### `.interval-wrapper`
**Type:** `<div>` container
**Purpose:** Wrapper for interval selector and label

**Responsive Behavior:**
- Mobile: Full width column layout
- Desktop: Inline layout

## CSS Classes Reference

### Utility Classes

#### `.hidden`
```css
.hidden {
  display: none;
}
```
**Usage:** Hide elements completely from layout

#### `.start` (Button State)
```css
#startButton.start {
  background-color: #28a745; /* Green */
}
```

#### `.stop` (Button State)
```css
#startButton.stop {
  background-color: #dc3545; /* Red */
}
```

#### `.switch-camera`
```css
.switch-camera {
  background-color: #007bff; /* Blue */
}
```

### Layout Classes

#### `.controls`
```css
.controls {
  display: flex;
  gap: 10px;
  align-items: center;
  /* ... additional styling */
}
```

#### `.io-areas`
```css
.io-areas {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  /* ... additional styling */
}
```

## Component Interactions

### State Management Flow

1. **Initialization**
   ```
   Page Load → Model Loading → Camera Access → Ready State
   ```

2. **Processing Cycle**
   ```
   Start Button → Disable Controls → Processing Loop → Stop Button → Enable Controls
   ```

3. **Camera Switching**
   ```
   Switch Button → Stop Current Stream → Request New Stream → Update UI
   ```

### Event Handling

#### Button Events
```javascript
startButton.addEventListener("click", () => {
  isProcessing ? handleStop() : handleStart();
});

switchCameraButton.addEventListener("click", switchCamera);
```

#### Window Events
```javascript
window.addEventListener("DOMContentLoaded", async () => {
  await initModel();
});

window.addEventListener("beforeunload", () => {
  if (stream) stream.getTracks().forEach(t => t.stop());
});
```

## Responsive Design

### Breakpoints

#### Mobile (`max-width: 600px`)
- Controls stack vertically
- Full-width buttons and selects
- Reduced padding and gaps
- Larger touch targets (16px font-size)

#### Desktop (Default)
- Horizontal control layout
- Compact spacing
- Smaller font sizes
- Hover effects enabled

### Responsive Components

#### Video Container
- Maintains aspect ratio across all screen sizes
- Maximum width constraint prevents oversizing
- Border adjusts for mobile (1px vs 2px)

#### Control Panel
- Flexible layout adapts to available space
- Interval wrapper becomes full-width on mobile
- Buttons expand to full width on mobile

#### Text Areas
- Font size increases on mobile for better readability
- Consistent padding across screen sizes
- Vertical resize only to prevent layout issues

## Accessibility Features

### Keyboard Navigation
- All interactive elements are focusable
- Tab order follows logical flow
- Enter key activates buttons

### Screen Reader Support
- Labels associated with form controls
- Descriptive button text
- Status updates in response textarea

### Visual Indicators
- Clear visual states for buttons
- Loading overlay with text indicator
- Color coding for different button states

### Mobile Accessibility
- Touch-friendly button sizes (minimum 44px)
- Sufficient color contrast
- Readable font sizes (16px minimum on mobile)