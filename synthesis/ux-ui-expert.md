# UX/UI Expert — Synthesis

## Overview

This panel covers 37 topics across 12 categories, providing a comprehensive reference for UX/UI design decisions in web development. Each topic follows a consistent structure: 5 fundamental principles, frameworks/methodologies, implementation patterns with CSS/HTML/JS code, common mistakes, and integration points.

## Category Map

### A. Foundational Principles — Interaction Design (4 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 01 | Design of Everyday Things | Affordance, signifiers, feedback, mapping |
| 02 | Goal-Directed Design | Personas and goal-based design process |
| 03 | First Principles of Interaction | 19 principles for interaction design |
| 04 | Microinteractions | Trigger, rules, feedback, loops design |

**Start here**: 01-Design of Everyday Things → 03-First Principles → 05-Usability Heuristics

### B. Usability and UX Research (5 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 05 | Usability Heuristics | Nielsen's 10 heuristics + evaluation method |
| 06 | Web Usability | Don't Make Me Think, satisficing, scanning |
| 07 | UX Research Methods | Qualitative vs quantitative, method selection |
| 08 | Psychology for Designers | Cognitive psychology applied to UI |
| 09 | Data-Driven UX | Metrics, analytics, A/B testing |

### C. UX Laws and Cognitive Psychology (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 10 | Laws of UX | Fitts, Hick, Jakob, Miller, Von Restorff, Doherty |
| 11 | Cognitive Load Theory | Intrinsic, extraneous, germane load reduction |
| 12 | Gestalt Principles | Proximity, similarity, closure, figure/ground |

### D. UX Strategy and Process (4 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 13 | Elements of UX | 5 planes: Strategy → Scope → Structure → Skeleton → Surface |
| 14 | Design Thinking | Empathize → Define → Ideate → Prototype → Test |
| 15 | Lean UX | Hypothesis-driven, Think-Make-Check, MVP |
| 16 | Jobs to Be Done | Functional/emotional/social jobs, outcome-driven |

### E. Information Architecture (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 17 | Information Architecture | Organization, labeling, navigation, search systems |
| 18 | UX Honeycomb | 7 facets: useful, usable, desirable, findable, accessible, credible, valuable |
| 19 | Sensemaking | 7 steps to structure information |

### F. Design Systems and Component Architecture (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 20 | Atomic Design | Atoms → molecules → organisms → templates → pages |
| 21 | Design Systems | Tokens, component libraries, documentation, governance |
| 22 | Material Design | Material principles, elevation, type scale, color system |

### G. Visual Design and Typography (4 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 23 | Data Visualization | Data-ink ratio, chart selection, graphical integrity |
| 24 | Web Typography | Type scale, readability, web fonts, fluid type |
| 25 | Visual Hierarchy | Grid systems, F/Z-pattern, whitespace, contrast |
| 26 | Color Theory | Color wheel, semantic colors, contrast ratios, palettes |

### H. Responsive and Mobile Design (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 27 | Responsive Web Design | Fluid grids, flexible images, media queries |
| 28 | Mobile First | Progressive enhancement, mobile constraints |
| 29 | Form Design | Validation, multi-step, mobile input optimization |

### I. Content Strategy (2 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 30 | Content Strategy | Content audit, editorial workflow, UX writing |
| 31 | Adaptive Content | Device-agnostic content, headless CMS, structured content |

### J. Accessibility (2 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 32 | WCAG Accessibility | POUR framework, conformance levels A/AA/AAA |
| 33 | Practical Accessibility | ARIA, semantic HTML, keyboard nav, screen readers |

### K. Emotional Design and Ethics (2 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 34 | Emotional Design | Hierarchy of user needs, 3 levels of emotional design |
| 35 | Design Ethics | Dark patterns, inclusive design, privacy by design |

### L. Modern Web Patterns (2 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 36 | Modern Web Patterns | Navigation, loading, scroll, modal, search patterns |
| 37 | Web Performance UX | Core Web Vitals, perceived performance, optimistic UI |

## Learning Paths

### Fundamentals Path
01 → 05 → 10 → 12 → 13
*Start with Norman's principles, then Nielsen's heuristics, UX laws, Gestalt, and the 5 planes.*

### Visual Design Path
23 → 24 → 25 → 26 → 20
*From data visualization through typography, hierarchy, color to component architecture.*

### UX Process Path
14 → 07 → 15 → 16 → 06
*Design Thinking, research methods, Lean UX, JTBD, then validate with usability testing.*

### Accessibility Path
32 → 33 → 35
*WCAG standards, practical implementation, then ethical design considerations.*

### Web Implementation Path
27 → 28 → 29 → 36 → 37
*Responsive design, mobile-first, forms, modern patterns, and performance.*

### Design System Path
20 → 21 → 22 → 26 → 24
*Atomic design, building systems, Material Design reference, color, typography.*

## Cross-Cutting Themes

1. **User-centricity** — Every decision should trace back to user needs and goals. Personas (02), research (07), JTBD (16), and testing (09) keep design grounded.
2. **Consistency** — Design systems (21), Gestalt principles (12), and Jakob's Law (10) all reinforce that consistency reduces cognitive load and builds trust.
3. **Progressive disclosure** — Show only what's needed at each step. Related to cognitive load (11), form design (29), and mobile-first (28).
4. **Accessibility-first** — Accessibility (32, 33) is not an afterthought but a foundation that benefits all users and improves overall UX quality.
5. **Performance is UX** — Web performance (37) directly impacts user experience. Core Web Vitals, perceived performance, and loading patterns (36) are design decisions.

## File Paths

All files are in `experts/ux-ui/` organized by subcategory. Each file is ~200-300 lines following the standard template.
