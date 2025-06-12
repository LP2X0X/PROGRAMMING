---
tags: js, keyword, mode
---

- When ECMAScript 5 (ES5) appeared, It added new features to the language and modified some of the existing ones. To keep the old code working, most such modifications are off by default. You need to explicitly enable them with a special directive: `"use strict"`.
- The directive looks like a string: `"use strict"` or `'use strict'`. When it is located at the top of a script, the whole script works the “modern” way.

```javascript
"use strict"; // Must be on top to take affect

// this code works the modern way
...
```

- There is no directive like `"no use strict"` that reverts the engine to old behavior.