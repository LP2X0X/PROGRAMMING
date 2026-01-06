---
tags: 
 - web
 - design
 - term
---

In web design, a **Toast** is a small, non-modal notification that provides feedback about an operation in a small popup. Unlike a modal, it is "passive"—it informs the user without blocking their workflow or requiring them to click a button to proceed.

The name comes from the way they "pop up" from the bottom or top of the screen, similar to toast popping out of a toaster.

---

## 1. Anatomy of a Toast

Toasts are designed to be peripheral and glanceable.

- **Icon:** Usually indicates the status (e.g., a green checkmark for success, a red "X" for error).
    
- **Message:** A short, punchy sentence (e.g., "Message Sent" or "Changes Saved").
    
- **Dismiss Button:** Often an "X", though many toasts are set to auto-dismiss after a few seconds ($3-5$ seconds).
    
- **Action (Optional):** Sometimes contains a simple link like "Undo."
    

---

## 2. When to Use a Toast

Toasts are best for **low-priority confirmations** that don't require the user to stop what they are doing.

- **Success confirmations:** "File uploaded successfully."
    
- **Background tasks:** "Exporting report... you'll be notified when it's ready."
    
- **Non-critical errors:** "Connection lost. Retrying..."
    
- **Undo actions:** "Item moved to trash. \[Undo]"
    

---

## 3. Best Practices for Design & UX

### A. Placement

Toasts should appear in a consistent location, typically the **top-right** or **bottom-center** of the viewport. On mobile, they are almost always at the bottom to stay within the "thumb zone."

### B. Duration

- **Short messages:** 2–3 seconds.
    
- **Complex messages (with actions):** 5–10 seconds.
    
- **Critical errors:** Should often stay visible until the user manually dismisses them.
    

### C. Accessibility 

Toasts can be difficult for screen reader users because they appear and disappear quickly.

- Use **`aria-live="polite"`** for standard notifications so the screen reader finishes its current sentence before announcing the toast.
    
- Use **`aria-live="assertive"`** only for critical errors that need immediate attention.
    

---

## 4. Toast vs. Modal vs. Snackbar

| **Feature**         | **Toast**      | **Modal**               | **Snackbar (Material Design)** |
| ------------------- | -------------- | ----------------------- | ------------------------------ |
| **Interruption**    | None           | High (Stops everything) | Low                            |
| **Auto-dismiss**    | Yes            | No                      | Yes                            |
| **Blocking**        | No (Non-modal) | Yes (Modal)             | No                             |
| **Typical Content** | Status update  | Decision/Forms          | Feedback + Action              |

---

## 5. Implementation Example (React)

While you can build them from scratch, most developers use libraries like **React Hot Toast** or **Sonner** because they handle "stacking" (preventing multiple toasts from overlapping) automatically.

```js
import toast, { Toaster } from 'react-hot-toast';

const notify = () => toast.success('Successfully toasted!');

function App() {
  return (
    <div>
      <button onClick={notify}>Make me a toast</button>
      <Toaster position="bottom-right" />
    </div>
  );
}
```