---
tags: react, technique, fundamental
---

In React, rendering a **list** means displaying multiple items dynamically from an **array** using `.map()`.

---

### ✅ Basic Example

```jsx
const skills = ['HTML', 'CSS', 'JavaScript'];

function SkillList() {
  return (
    <ul>
      {skills.map((skill, index) => (
        <li key={index}>{skill}</li>
      ))}
    </ul>
  );
}
```

* Use `.map()` to loop through the array.
* Always provide a **`key`** prop (preferably a unique ID).
* Wrap list items in a parent element (like `<ul>` or `<div>`).

---

### ✅ With Components

```jsx
function Skill({ name }) {
  return <li>{name}</li>;
}

function SkillList() {
  const skills = ['HTML', 'CSS', 'JavaScript'];

  return (
    <ul>
      {skills.map((skill, index) => (
        <Skill key={index} name={skill} />
      ))}
    </ul>
  );
}
```

---

### ✅ With Objects

```jsx
const skills = [
  { id: 1, name: 'React' },
  { id: 2, name: 'Git' },
  { id: 3, name: 'Figma' },
];

function SkillList() {
  return (
    <ul>
      {skills.map(skill => (
        <li key={skill.id}>{skill.name}</li>
      ))}
    </ul>
  );
}
```

---

### ⚠️ Key Tips

* Use a **unique `key`** for each item (like an ID, not the array index if avoidable).
* Avoid modifying the original array in `.map()`.
* You can also add conditionals and styles inside `.map()`.