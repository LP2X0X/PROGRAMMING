---
tags: 
 - regex
 - character 
 - class
---

- In some cases, we might know that there are specific characters that we don't want to match too, for example, we might only want to match phone numbers that are _not_ from the area code 650.

- To represent this, we use a similar expression that _excludes specific characters_ using the _square brackets_ and the _^_ (_hat_). For example, the pattern \[^abc\] will match any _single_ character _except for_ the letters a, b, or c. 

- Remember, a negated character class means 'match a character that's not listed' and not 'don't match what is listed'. A convenient way to view a negated class is that it is simply a **shorthand for a normal class that includes all possible characters except those that are listed**.

```ad-example
Pattern: q\[\^u]
Matches: qa, qb
Not matches: Sq, tq
```