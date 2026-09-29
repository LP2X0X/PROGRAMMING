---
tags:
  - css
  - javascript
  - animation
  - frontend
---

## CSS vs JS — When to Use Which

CSS handles a lot of animation on its own — it's not just for static styling.

### What CSS Can Do

- Transitions (hover, state changes)
- Keyframe animations (spin, pulse, slide)
- Scroll-driven animations (new CSS feature)
- Transforms (rotate, scale, translate, skew)
- These run on the **GPU** — very performant

### When JS Is Needed

- Animation depends on user input (mouse position, scroll position)
- Complex sequencing (step 1 finishes → step 2 starts → step 3...)
- Dynamic values (animate to a calculated position)
- Physics-based (spring, bounce with real momentum)
- Orchestrating multiple elements together

### Decision Table

| Effect                    | CSS | JS          |
| ------------------------- | --- | ----------- |
| Hover effects             | Yes | Overkill    |
| Loading spinners          | Yes | Overkill    |
| Fade in/out               | Yes | Overkill    |
| Parallax on scroll        | Can | Easier w/JS |
| Mouse-following cursor    | No  | Yes         |
| Staggered list animations | Hacky | Clean     |
| Timeline sequences        | No  | Yes         |
| Physics/spring            | No  | Yes         |

**Rule of thumb:** CSS is the first choice for animation. Reach for JS only when CSS can't express the logic you need.

---

## Resources for Learning Advanced Frontend Effects

### Inspiration

- **Awwwards** (awwwards.com) — award-winning websites with advanced effects
- **CodePen** (codepen.io) — search any effect, see the code immediately
- **Codrops** (tympanus.net/codrops) — tutorials with advanced CSS/JS effects and demos

### Learning

- **GSAP** (gsap.com) — industry standard animation library; docs cover scroll animations, morphing, parallax
- **Framer Motion** docs — for React projects
- **CSS Tricks** (css-tricks.com) — deep articles on CSS animations, transforms, clip-path tricks
- **Kevin Powell** (YouTube) — advanced CSS effects explained clearly
- **Hyperplexed** (YouTube) — recreates cool website effects step by step

### Libraries and Tools by Effect Type

| Effect             | Look into                          |
| ------------------ | ---------------------------------- |
| Scroll animations  | GSAP ScrollTrigger, Intersection Observer |
| Page transitions   | Barba.js, View Transitions API     |
| 3D effects         | Three.js, CSS perspective/transform |
| Particle effects   | tsParticles, Canvas API            |
| Text animations    | SplitType + GSAP                   |
| Cursor effects     | Custom cursor with JS + CSS        |
| Parallax           | Lenis + GSAP                       |
| Morphing shapes    | SVG + GSAP MorphSVG                |

> [!tip]
> CodePen is the best starting point — find an effect you like, fork it, break it apart.
