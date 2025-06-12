---
tags: js, miscellaneous
---

- snake_case and camelCase.
- There are two kind of constant:
	- true constant which have values that will never change whenever you run the program. It's customary to use all-caps snake_case like HOUR_IN_A_DAY.
	- Value you've made constant because you don't want to accidentally change them in your code. For this type, you should use the same convention as for variables.
- By placing the script element at the end of the body, we can be sure that all the page content has been loaded into the DOM before we run our JavaScript since Anytime the browser reaches a script element, it executes the whole script before continuing.
- Any HTML between the opening and closing tags will appear only if the browser doesn’t support the canvas element, so this can be used as a fallback for older or text-only browsers.
