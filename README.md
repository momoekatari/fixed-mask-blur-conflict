# fixed-mask-blur-conflict
A generic, clean solution for resolving rendering conflicts between scrolling backdrop-filter elements and fixed overlay masks in web interfaces.

# Web UI: Resolving Conflict Between Fixed Overlay Masks and Scrolling `backdrop-filter` Elements

## 1. The Problem
In modern web interfaces, a common layout involves a scrollable content area containing elements with `backdrop-filter: blur()`, alongside a fixed top overlay mask to hide content scrolling beneath it. 

When a glassmorphism element scrolls under the mask, the pixel background changes. Because `backdrop-filter` blurs the *actual pixels* behind the element, the blur result becomes unpredictable, causing the glass element to "flicker", "jump", or suddenly change color.

## 2. Root Cause
The conflict arises from the mask layer itself. If the fixed mask is implemented as a semi-transparent color block (`background: rgba(...)` or `linear-gradient`), the pixels behind the glass element change to `[background + color block]` when scrolling into the mask zone. The input to the blur changes, so the output changes.

The solution is not merely layering—it requires the mask's background to perfectly align with the underlying true background at the pixel level.

## 3. The Solution

### 3.1 Key 1: The Mask Must Use the Background Image, Not a Color
Sync the mask's `background-image` with the actual underlying background. Use `background-attachment: fixed` to ensure the image aligns pixel-perfectly with the base layer.

```css
.top-mask {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 90px;
    z-index: 5;
    background-image: url('your-global-background.jpg'); /* Must be the same image */
    background-size: cover;
    background-position: center top;
    background-attachment: fixed; /* Crucial for pixel alignment */
}
```

### 3.2 Key 2: Use `mask-image` for the Fade-out Effect, Not a Gradient Background
Using `background: linear-gradient()` creates a "faded solid color block." Using `mask-image` acts as a true transparency crop on the layer. The masked content fades authentically rather than being obscured by a gradient overlay.

```css
.top-mask {
    /* 0-60px fully opaque, 60-90px fading out */
    -webkit-mask-image: linear-gradient(to bottom, black 0, black 60px, transparent 90px);
    mask-image: linear-gradient(to bottom, black 0, black 60px, transparent 90px);
}
```

### 3.3 Key 3: If the Base Background has an Overlay, the Mask Must Match It
If the base background has a global dimming or tint overlay (e.g., for readability), the mask must replicate this overlay to prevent the top edge from appearing brighter.

```css
.top-mask::before {
    content: '';
    position: absolute;
    inset: 0;
    background: var(--overlay-color);
    opacity: var(--overlay-opacity);
    pointer-events: none;
    /* The parent's mask-image automatically applies to ::before, keeping edges seamless */
}
```

## 4. Complete Implementation
```css
.top-mask {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 90px;
    z-index: 5;
    pointer-events: none;
    background-image: url('background.jpg');
    background-size: cover;
    background-position: center top;
    background-attachment: fixed;
    -webkit-mask-image: linear-gradient(to bottom, black 0, black 60px, transparent 90px);
    mask-image: linear-gradient(to bottom, black 0, black 60px, transparent 90px);
}

.top-mask::before {
    content: '';
    position: absolute;
    inset: 0;
    background: var(--overlay-color);
    opacity: var(--overlay-opacity);
    pointer-events: none;
}
```

## 5. Implementation Notes
- **iOS Safari**: `background-attachment: fixed` may cause minor jitter in certain iOS versions. Apply feature detection or fallback mechanisms if strictly necessary.
- **High-Frequency Textures**: If the global background is a high-frequency texture (e.g., fine grids), exact pixel alignment of `background-position` between the mask and base layer is critical to avoid visible seams.
- **Pointer Events**: Always apply `pointer-events: none` to the mask layer to prevent it from blocking interactions below.

## 6. Use Cases
This approach is applicable to any layout involving fixed headers and scrollable areas containing `backdrop-filter` elements:
- Fixed header overlay + scrolling list of glassmorphism cards.
- Sticky media player + translucent cards scrolling underneath.
- Any fixed top bar interacting with dynamic scrolling transparent elements.
