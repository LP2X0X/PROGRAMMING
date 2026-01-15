---
tags: 
 - react
 - prop
 - event
---

## `onError` in React — Detailed Notes

### What `onError` Is

`onError` is a **React event handler** that fires when a **resource fails to load** or when a **media element encounters an error**.

It is **not** a general JavaScript error handler and **does not catch render errors**.

---

## Elements That Support `onError`

`onError` works on **specific HTML elements**, primarily those that load external resources:

### Common Supported Elements

- `<img>`
    
- `<video>`
    
- `<audio>`
    
- `<source>`
    
- `<link>`
    
- `<script>`
    
- `<iframe>`
    

### Example

```tsx
<img
  src="/poster.jpg"
  alt="Movie poster"
  onError={() => {
    console.error("Image failed to load");
  }}
/>
```

---

## What Triggers `onError`

- Image fails to load
    
- Network failure
    
- Invalid URL
    
- Unsupported media format
    
- Script load failure
    

---

## What `onError` Does NOT Catch

❌ JavaScript runtime errors  
❌ React render errors  
❌ Errors inside event handlers  
❌ Errors inside effects

For those, you must use **Error Boundaries**.

---

## Event Type in TypeScript

The event type depends on the element.

### Image Example

```ts
const handleError = (
  e: React.SyntheticEvent<HTMLImageElement, Event>
) => {
  console.log(e.currentTarget.src);
};
```

Use `currentTarget`, not `target`, for correct typing.

---

## Common Patterns

### 1. Fallback Image

```tsx
<img
  src={posterUrl}
  onError={(e) => {
    e.currentTarget.src = "/fallback.png";
  }}
/>
```

---

### 2. Conditional Rendering

```tsx
const [hasError, setHasError] = useState(false);

<img
  src={url}
  onError={() => setHasError(true)}
/>

{hasError && <Placeholder />}
```

---

### 3. Media Error Handling

```tsx
<video
  src={videoUrl}
  controls
  onError={(e) => {
    console.log(e.currentTarget.error);
  }}
/>
```

---

## `onError` vs Error Boundaries

|Feature|`onError`|Error Boundary|
|---|---|---|
|Asset loading errors|✅|❌|
|Render errors|❌|✅|
|JS runtime errors|❌|✅|
|Component tree recovery|❌|✅|

---

## Important React-Specific Details

### Synthetic Events

- `onError` uses React’s Synthetic Event system
    
- Event object is pooled (React <17) — do not async access
    

---

### `currentTarget` vs `target`

Always prefer:

```ts
e.currentTarget
```

Because:

- It is correctly typed
    
- Always refers to the element with the handler
    

---

## Common Mistakes

❌ Expecting `onError` to catch component crashes  
❌ Using it on unsupported elements  
❌ Accessing `e.target.src` (unsafe typing)

---

## One-Line Summary

> **`onError` handles resource-loading failures on specific elements, not JavaScript or React rendering errors.**