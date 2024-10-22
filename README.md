# P5.js Camera Filter Development Guide

[![Watch the video](https://img.youtube.com/vi/bDqbez7nRkA/0.jpg)](https://www.youtube.com/watch?v=bDqbez7nRkA)

 ^ Demo Video

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Pixel Array Structure](#pixel-array)
3. [Common Techniques](#techniques)
4. [Performance Optimization](#performance)
5. [Example Filters](#examples)
6. [Advanced Topics](#advanced)
7. [Troubleshooting](#troubleshooting)

## Fundamentals

### Available Variables
In your filter code, you have access to these variables:
- `pixels[]`: The pixel array containing image data
- `width`: Canvas width in pixels
- `height`: Canvas height in pixels

### Pixel Array Structure
Each pixel uses 4 array positions (RGBA):
```javascript
pixels[i]     // Red (0-255)
pixels[i+1]   // Green (0-255)
pixels[i+2]   // Blue (0-255)
pixels[i+3]   // Alpha (0-255, usually 255)
```

### Basic Index Calculation
For any (x,y) coordinate:
```javascript
let index = (y * width + x) * 4;
```

## Working with Pixels

### Color Manipulation
```javascript
// Basic color getting
let r = pixels[index];
let g = pixels[index + 1];
let b = pixels[index + 2];

// Color setting
pixels[index] = newR;
pixels[index + 1] = newG;
pixels[index + 2] = newB;
```

### Common Color Operations
```javascript
// Brightness
let brightness = (r + g + b) / 3;

// Grayscale
pixels[index] = brightness;
pixels[index + 1] = brightness;
pixels[index + 2] = brightness;

// Invert
pixels[index] = 255 - r;
pixels[index + 1] = 255 - g;
pixels[index + 2] = 255 - b;
```

## Common Techniques

### 1. Pixel-by-Pixel Processing
```javascript
for (let i = 0; i < pixels.length; i += 4) {
    // Process each pixel
    let r = pixels[i];
    let g = pixels[i + 1];
    let b = pixels[i + 2];
    
    // Modify and write back
    pixels[i] = newR;
    pixels[i + 1] = newG;
    pixels[i + 2] = newB;
}
```

### 2. Coordinate-Based Processing
```javascript
for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
        let index = (y * width + x) * 4;
        // Process pixel at (x,y)
    }
}
```

### 3. Block Processing
```javascript
let blockSize = 10;
for (let y = 0; y < height; y += blockSize) {
    for (let x = 0; x < width; x += blockSize) {
        // Process block of pixels
        for (let dy = 0; dy < blockSize; dy++) {
            for (let dx = 0; dx < blockSize; dx++) {
                let index = ((y + dy) * width + (x + dx)) * 4;
                // Modify pixels
            }
        }
    }
}
```

## Example Filters

### 1. Basic Color Tint
```javascript
// Red tint effect
for (let i = 0; i < pixels.length; i += 4) {
    pixels[i] = pixels[i] * 1.5;     // Boost red
    pixels[i + 1] *= 0.7;            // Reduce green
    pixels[i + 2] *= 0.7;            // Reduce blue
}
```

### 2. Dynamic Wave Distortion
```javascript
let temp = pixels.slice(); // Create copy of pixels
let time = Date.now() * 0.001; // Current time in seconds

for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
        // Create wave effect
        let offset = Math.sin(x * 0.05 + time) * 10;
        
        // Calculate source and target positions
        let sourceY = Math.floor(y + offset);
        sourceY = Math.max(0, Math.min(height - 1, sourceY));
        
        let targetIndex = (y * width + x) * 4;
        let sourceIndex = (sourceY * width + x) * 4;
        
        // Copy pixels with wave distortion
        pixels[targetIndex] = temp[sourceIndex];
        pixels[targetIndex + 1] = temp[sourceIndex + 1];
        pixels[targetIndex + 2] = temp[sourceIndex + 2];
    }
}
```

### 3. Advanced Glitch Effect
```javascript
// Parameters
let glitchIntensity = 0.1;
let blockSize = 20;
let rgbShiftAmount = 5;

// Create temporary copy
let temp = pixels.slice();

// Process blocks
for (let y = 0; y < height; y += blockSize) {
    for (let x = 0; x < width; x += blockSize) {
        // Random glitch decision
        if (Math.random() < glitchIntensity) {
            // RGB shift for glitch blocks
            let shiftX = Math.floor(Math.random() * rgbShiftAmount);
            
            for (let dy = 0; dy < blockSize && y + dy < height; dy++) {
                for (let dx = 0; dx < blockSize && x + dx < width; dx++) {
                    let sourceX = x + dx + shiftX;
                    sourceX = Math.min(width - 1, Math.max(0, sourceX));
                    
                    let targetIndex = ((y + dy) * width + (x + dx)) * 4;
                    let sourceIndex = ((y + dy) * width + sourceX) * 4;
                    
                    // Shift color channels
                    pixels[targetIndex] = temp[sourceIndex + 2]; // R gets B
                    pixels[targetIndex + 1] = temp[sourceIndex]; // G gets R
                    pixels[targetIndex + 2] = temp[sourceIndex + 1]; // B gets G
                }
            }
        }
    }
}

// Add scan lines
for (let y = 0; y < height; y += 3) {
    for (let x = 0; x < width; x++) {
        let index = (y * width + x) * 4;
        pixels[index] *= 0.7;     // Darken R
        pixels[index + 1] *= 0.7; // Darken G
        pixels[index + 2] *= 0.7; // Darken B
    }
}
```

## Performance Tips

1. **Minimize Array Copies**
   - Only create temp arrays when necessary
   - Reuse temporary arrays when possible

2. **Use Efficient Loops**
   ```javascript
   // More efficient
   for (let i = 0; i < pixels.length; i += 4) {
       // Process directly
   }
   
   // Less efficient
   pixels.forEach((value, index) => {
       // Process with callback
   });
   ```

3. **Optimize Math Operations**
   ```javascript
   // More efficient
   let val = x * 4;
   
   // Less efficient
   let val = Math.floor(x * 4);
   ```

4. **Minimize Conditionals in Loops**
   ```javascript
   // Better
   let condition = x < threshold;
   for (let i = 0; i < pixels.length; i += 4) {
       if (condition) {
           // Process
       }
   }
   
   // Worse
   for (let i = 0; i < pixels.length; i += 4) {
       if (x < threshold) {
           // Process
       }
   }
   ```

## Advanced Techniques

### 1. Kernel Convolution
For effects like blur, sharpen, edge detection:
```javascript
function applyKernel(kernel) {
    let temp = pixels.slice();
    let kSize = Math.sqrt(kernel.length);
    let kOffset = Math.floor(kSize/2);
    
    for (let y = kOffset; y < height - kOffset; y++) {
        for (let x = kOffset; x < width - kOffset; x++) {
            let r = 0, g = 0, b = 0;
            
            // Apply kernel
            for (let ky = 0; ky < kSize; ky++) {
                for (let kx = 0; kx < kSize; kx++) {
                    let ix = x + kx - kOffset;
                    let iy = y + ky - kOffset;
                    let iidx = (iy * width + ix) * 4;
                    let kidx = ky * kSize + kx;
                    
                    r += temp[iidx] * kernel[kidx];
                    g += temp[iidx + 1] * kernel[kidx];
                    b += temp[iidx + 2] * kernel[kidx];
                }
            }
            
            let idx = (y * width + x) * 4;
            pixels[idx] = r;
            pixels[idx + 1] = g;
            pixels[idx + 2] = b;
        }
    }
}

// Example kernels
const BLUR_KERNEL = [
    1/9, 1/9, 1/9,
    1/9, 1/9, 1/9,
    1/9, 1/9, 1/9
];

const SHARPEN_KERNEL = [
     0, -1,  0,
    -1,  5, -1,
     0, -1,  0
];
```

### 2. Color Space Conversions
```javascript
function rgbToHsl(r, g, b) {
    r /= 255; g /= 255; b /= 255;
    let max = Math.max(r, g, b);
    let min = Math.min(r, g, b);
    let h, s, l = (max + min) / 2;

    if (max === min) {
        h = s = 0;
    } else {
        let d = max - min;
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
        switch (max) {
            case r: h = (g - b) / d + (g < b ? 6 : 0); break;
            case g: h = (b - r) / d + 2; break;
            case b: h = (r - g) / d + 4; break;
        }
        h /= 6;
    }
    return [h, s, l];
}
```

## Troubleshooting

### Common Issues

1. **Black Screen**
   - Check if you're accessing pixels out of bounds
   - Ensure you're not setting invalid color values

2. **Performance Issues**
   - Reduce nested loops
   - Minimize array copies
   - Use smaller block sizes for block processing

3. **Artifacts**
   - Check boundary conditions
   - Ensure proper index calculations
   - Validate color value ranges (0-255)

### Debug Tips
```javascript
// Add debug logging
let debugIndex = (height/2 * width + width/2) * 4;
console.log('Center pixel:', {
    r: pixels[debugIndex],
    g: pixels[debugIndex + 1],
    b: pixels[debugIndex + 2]
});

// Test specific regions
for (let y = 0; y < 10; y++) {
    for (let x = 0; x < 10; x++) {
        let index = (y * width + x) * 4;
        // Test processing
    }
}
```

Would you like me to provide more specific examples or elaborate on any particular section?
