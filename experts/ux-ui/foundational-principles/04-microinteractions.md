# Microinteractions

## Profile

### What It Is
Microinteractions are contained product moments that revolve around a single use case — a small piece of functionality that does one thing well. Dan Saffer's framework decomposes every microinteraction into four parts: triggers, rules, feedback, and loops and modes. While individually subtle, microinteractions collectively define the feel of a digital product. They are the difference between an interface that feels polished and alive versus one that feels flat and mechanical. In web development, they are primarily implemented through CSS transitions, animations, and lightweight JavaScript.

### Origin and Evolution
- **2005-2010** — The rise of rich web applications (Gmail, Google Maps) demonstrates that small interaction details profoundly affect perceived quality
- **2013** — Dan Saffer publishes *Microinteractions: Designing with Details*, codifying the four-part structure and establishing microinteractions as a design discipline
- **2014-2016** — CSS transitions and animations become widely supported, making microinteractions accessible without JavaScript frameworks
- **2016-present** — The Web Animations API, CSS custom properties, and frameworks like Framer Motion lower the barrier further, enabling sophisticated microinteractions
- **2020s** — The `prefers-reduced-motion` media query becomes standard practice, making motion design accessible

### Key Authors and References
- **Dan Saffer** — *Microinteractions: Designing with Details* (2013)
- **Val Head** — *Designing Interface Animation* (2016)
- **Rachel Nabors** — *Animation at Work* (2017)
- **Material Design Team (Google)** — Motion Design Guidelines (2014-present)
- **Apple Human Interface Guidelines** — Motion section (updated regularly)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Trigger | The event that initiates a microinteraction (user action or system condition) |
| Rules | The logic that determines what happens once the microinteraction is triggered |
| Feedback | The sensory response (visual, auditory, haptic) that communicates the rules |
| Loops | How the microinteraction changes over repeated use or over time |
| Modes | Alternate states within the microinteraction that change the rules |
| Manual Trigger | User-initiated (click, hover, swipe, voice command) |
| System Trigger | Condition-initiated (new message, time elapsed, location change) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Details signal quality** — Users cannot articulate why one product feels better than another, but they can feel it. Microinteractions are the primary carriers of perceived quality and craftsmanship
2. **Feedback makes systems feel alive** — Every user action should produce a proportional, immediate response. Silence from an interface creates anxiety; appropriate feedback creates confidence
3. **Rules must be invisible** — The logic governing a microinteraction should feel natural and obvious. If users notice the rules, the rules are too complex or too surprising
4. **Restraint over spectacle** — A microinteraction that draws attention to itself is a failed microinteraction. The goal is to support the user's task, not to showcase animation skill
5. **Respect user preferences and context** — Motion that delights at first may annoy on the hundredth repetition. Provide reduced-motion alternatives, respect system settings, and let loops evolve over time

### When to Use vs. When to Avoid

**Use when:**
- Providing immediate feedback for user actions (button clicks, form submissions, toggles)
- Communicating state changes that would otherwise be invisible (data saved, message sent, error occurred)
- Guiding attention to new or changed content (notifications, inline validation, content transitions)
- Making wait times feel shorter through animated progress indicators
- Reinforcing the spatial model of the interface (where elements come from and go to)

**Avoid over-application when:**
- Animation duration would slow down a user who is performing rapid, repetitive actions
- The user has indicated a preference for reduced motion (via OS settings or app preference)
- Decorative motion adds no informational value and distracts from content
- Performance-constrained environments (low-powered devices, slow connections) where animation causes jank
- Critical error states where the user needs to focus on reading, not watching animation

---

## Frameworks and Methodologies

### 1. Saffer's Four-Part Structure — design framework
1. **Identify the trigger** — What initiates the interaction? Is it a user action (manual trigger: click, hover, gesture) or a system event (system trigger: timer, data change, threshold reached)?
2. **Define the rules** — What happens once triggered? Rules determine the sequence of events, the duration, and the conditions for completion. Keep rules minimal and predictable
3. **Design the feedback** — How does the user know what happened? Choose the appropriate feedback channel: visual (animation, color change), auditory (sound), or haptic (vibration). Visual feedback is the default for web
4. **Consider loops and modes** — How does the interaction behave over time? Does feedback persist? Does it change on repeated use? Are there alternate modes (e.g., a toggle that switches between play/pause)?

### 2. Motion Audit — for existing interfaces
1. Catalog every animation and transition in the product
2. For each, identify: What triggers it? What rule does it follow? What feedback does it provide?
3. Classify: Is it functional (communicates state) or decorative (adds personality)?
4. Test: Does it still serve a purpose on the 100th viewing? Does it work with reduced motion?
5. Remove or simplify any motion that fails steps 3-4

---

## How to Apply

### Decision Checklist
- [ ] Does every interactive element provide visual feedback within 100ms of user action?
- [ ] Can the user understand the system's current state from the feedback alone?
- [ ] Are animations short enough (150-300ms for most transitions) to feel responsive rather than slow?
- [ ] Have you implemented `prefers-reduced-motion` alternatives for all animations?
- [ ] Does each microinteraction have a clear trigger, rule, feedback, and loop/mode consideration?
- [ ] Are decorative animations distinguishable from functional ones in your codebase?
- [ ] Do animations run at 60fps without causing layout shifts or jank?
- [ ] Have you tested animations on low-powered devices?

### Implementation Patterns

**Button microinteraction — complete trigger/feedback cycle:**
```css
.btn {
  position: relative;
  padding: 12px 28px;
  background-color: #2563eb;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  overflow: hidden;
  transition: background-color 0.15s ease, transform 0.1s ease;
}

/* Trigger: hover — Feedback: subtle color shift */
.btn:hover {
  background-color: #1d4ed8;
}

/* Trigger: mousedown — Feedback: press-in effect */
.btn:active {
  transform: scale(0.97);
}

/* Trigger: focus — Feedback: visible focus ring (keyboard users) */
.btn:focus-visible {
  outline: 3px solid #93c5fd;
  outline-offset: 2px;
}

/* Ripple effect on click (feedback for touch/click trigger) */
.btn::after {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(circle, rgba(255,255,255,0.3) 10%, transparent 10%);
  background-size: 0;
  background-position: center;
  background-repeat: no-repeat;
  transition: background-size 0.4s ease;
}

.btn:active::after {
  background-size: 300%;
  transition: background-size 0s;
}

/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  .btn {
    transition: none;
  }
  .btn::after {
    display: none;
  }
}
```

**Form validation — inline feedback microinteraction:**
```html
<div class="form-field">
  <label for="email">Email</label>
  <input
    type="email"
    id="email"
    class="input"
    required
    aria-describedby="email-feedback"
  />
  <span id="email-feedback" class="feedback" role="status" aria-live="polite"></span>
</div>
```

```css
.input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #d1d5db;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

/* Trigger: focus — Feedback: highlight border */
.input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);
  outline: none;
}

/* Trigger: valid input — Feedback: green border + checkmark */
.input:valid:not(:placeholder-shown) {
  border-color: #16a34a;
}

/* Trigger: invalid input after interaction — Feedback: red border */
.input:invalid:not(:placeholder-shown):not(:focus) {
  border-color: #dc2626;
}

.feedback {
  display: block;
  min-height: 1.25rem;
  font-size: 0.8125rem;
  margin-top: 4px;
  transition: opacity 0.2s ease;
}

.feedback--error { color: #dc2626; }
.feedback--success { color: #16a34a; }

@media (prefers-reduced-motion: reduce) {
  .input, .feedback { transition: none; }
}
```

**Toggle switch — mode microinteraction:**
```html
<label class="toggle" role="switch" aria-checked="false" tabindex="0">
  <input type="checkbox" class="toggle__input" />
  <span class="toggle__track">
    <span class="toggle__thumb"></span>
  </span>
  <span class="toggle__label">Dark mode</span>
</label>
```

```css
.toggle { display: inline-flex; align-items: center; gap: 12px; cursor: pointer; }
.toggle__input { position: absolute; opacity: 0; width: 0; height: 0; }

.toggle__track {
  position: relative; width: 52px; height: 28px;
  background-color: #d1d5db; border-radius: 14px;
  transition: background-color 0.25s ease;
}
.toggle__thumb {
  position: absolute; top: 2px; left: 2px; width: 24px; height: 24px;
  background-color: #fff; border-radius: 50%;
  box-shadow: 0 1px 3px rgba(0,0,0,0.2);
  transition: transform 0.25s cubic-bezier(0.4,0,0.2,1);
}

/* Trigger: checked — Rule: thumb slides right, track turns blue */
.toggle__input:checked + .toggle__track { background-color: #2563eb; }
.toggle__input:checked + .toggle__track .toggle__thumb { transform: translateX(24px); }
.toggle__input:focus-visible + .toggle__track { outline: 3px solid #93c5fd; outline-offset: 2px; }

@media (prefers-reduced-motion: reduce) {
  .toggle__track, .toggle__thumb { transition-duration: 0.01ms; }
}
```

**Loading state — staggered fade-in and spinner:**
```css
/* Trigger: content loaded — Feedback: fade-in with stagger */
.card {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.card--entering { opacity: 0; transform: translateY(8px); }
.card-list .card:nth-child(1) { transition-delay: 0ms; }
.card-list .card:nth-child(2) { transition-delay: 60ms; }
.card-list .card:nth-child(3) { transition-delay: 120ms; }

/* Indeterminate spinner */
.spinner {
  width: 24px; height: 24px;
  border: 3px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

@media (prefers-reduced-motion: reduce) {
  .card { transition: none; }
  .spinner { animation: none; }
}
```

**Notification toast — system trigger with auto-dismiss loop:**
```css
.toast {
  position: fixed; bottom: 24px; right: 24px;
  padding: 16px 24px; background: #1f2937; color: #fff;
  border-radius: 8px; box-shadow: 0 8px 24px rgba(0,0,0,0.2);
  transform: translateY(100px); opacity: 0;
  transition: transform 0.3s cubic-bezier(0.4,0,0.2,1), opacity 0.3s ease;
}
.toast--visible { transform: translateY(0); opacity: 1; }

/* Loop: auto-dismiss progress bar shrinks over 4 seconds */
.toast__progress {
  position: absolute; bottom: 0; left: 0; height: 3px;
  background-color: #60a5fa; border-radius: 0 0 8px 8px;
  animation: shrink 4s linear forwards;
}
@keyframes shrink { from { width: 100%; } to { width: 0%; } }

@media (prefers-reduced-motion: reduce) {
  .toast { transition: none; transform: translateY(0); }
  .toast__progress { animation: none; width: 100%; }
}
```

### Common Mistakes
1. **Animation for animation's sake** — Adding bouncing, spinning, or sliding effects that do not communicate any state change. Every animation should answer the question "what information does this convey?"
2. **Ignoring prefers-reduced-motion** — Approximately 25% of users have motion sensitivity to some degree. Failing to provide reduced-motion alternatives is both an accessibility failure and a usability failure
3. **Slow transitions that block interaction** — Animations longer than 300ms for common interactions (button press, tab switch) make the interface feel sluggish. Reserve longer durations (400-600ms) for large-scale transitions like page changes
4. **Inconsistent easing** — Mixing linear, ease-in, ease-out, and cubic-bezier curves randomly across the product creates a disjointed feel. Establish a motion system with 2-3 standard easing curves
5. **Forgetting the loop** — Designing only the first occurrence of a microinteraction. Consider: does this animation add value on the 50th repetition, or does it become irritating? Subtle is sustainable

### Integration with Other Topics
- **36-modern-web-patterns** — Modern web patterns like skeleton screens, optimistic UI, and pull-to-refresh are composed of microinteractions. Saffer's four-part framework helps design each pattern's feedback loop
- **37-web-performance-ux** — Poorly implemented microinteractions (layout-triggering animations, unthrottled JS-driven motion) directly harm performance. Use CSS transforms and opacity for 60fps animations
- **34-emotional-design** — Microinteractions are the primary vehicle for Norman's visceral and behavioral emotional design layers. A well-crafted button press communicates care; a janky transition communicates neglect
- **01-design-of-everyday-things** — Microinteractions are how feedback (one of Norman's six principles) is actually delivered in digital products. Every visual response to a user action is a microinteraction

---

## Resources

### Essential Reading
- Saffer, D. (2013). *Microinteractions: Designing with Details*. O'Reilly Media.
- Head, V. (2016). *Designing Interface Animation*. Rosenfeld Media.
- Nabors, R. (2017). *Animation at Work*. A Book Apart.

### Online Resources
- [Material Design Motion Guidelines](https://m3.material.io/styles/motion/overview) — Google's comprehensive motion system with duration, easing, and pattern references
- [MDN: CSS Transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions) — Technical reference for implementing web microinteractions
- [MDN: prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) — Accessibility guide for motion
- [Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) — For complex, JavaScript-controlled animations

### Practice Exercises
1. **Microinteraction decomposition**: Pick five microinteractions in a product you use daily (e.g., liking a post, toggling a setting, submitting a form). For each, identify the trigger, rules, feedback, and loop/mode
2. **Button state audit**: Create a single button component with complete microinteraction coverage: default, hover, focus-visible, active, loading, success, error, and disabled states. Ensure all states have appropriate visual feedback and reduced-motion alternatives
3. **Motion system definition**: Define a motion system for a project with exactly three easing curves (standard, enter, exit) and three duration tokens (instant: 100ms, standard: 200ms, expressive: 400ms). Apply these consistently to every transition in the project
4. **Reduced motion refactoring**: Take an existing page with animations and add complete `prefers-reduced-motion` support. Replace motion-based feedback with instant state changes, opacity changes, or static visual indicators. Test with macOS/Windows reduced motion enabled
