# Web Performance UX

## Profile

### What It Is
Web performance UX is the discipline of understanding and optimizing how users perceive speed, responsiveness, and visual stability of web interfaces. It goes beyond raw metrics and server response times to address the human experience of waiting, interacting, and consuming content. A page that loads in 2 seconds but shows nothing until it fully renders feels slower than a page that takes 3 seconds but progressively reveals meaningful content from the first 500 milliseconds.

### Origin and Evolution
- **2009** — Steve Souders publishes *Even Faster Web Sites*, establishing front-end performance as a discipline separate from back-end optimization
- **2012** — Google begins factoring page speed into search rankings, making performance a business priority
- **2015** — The RAIL performance model (Response, Animation, Idle, Load) is introduced by Google Chrome team, framing performance around user perception
- **2018** — First Contentful Paint (FCP) and Time to Interactive (TTI) become standard lab metrics; Lighthouse audits gain widespread adoption
- **2020** — Google introduces Core Web Vitals (LCP, FID, CLS) as unified, user-centric field metrics tied to search ranking
- **2023** — Interaction to Next Paint (INP) replaces First Input Delay as the responsiveness metric, better capturing real-world interaction latency
- **2024–present** — Addy Osmani's work on JavaScript performance and Aurora project (Next.js, Angular) brings framework-level performance defaults; speculation rules and View Transitions API enable instant navigation patterns

### Key Authors and References
- **Addy Osmani** — *Learning Patterns* (2022), performance optimization work at Google Chrome
- **Steve Souders** — *High Performance Web Sites* (2007), *Even Faster Web Sites* (2009)
- **Ilya Grigorik** — *High Performance Browser Networking* (2013, O'Reilly)
- **Philip Walton** — Core Web Vitals definitions and measurement methodology (web.dev)
- **Google Chrome Team** — RAIL model, Core Web Vitals, web.dev documentation

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| LCP (Largest Contentful Paint) | Time until the largest visible content element renders; target < 2.5 seconds |
| INP (Interaction to Next Paint) | Latency from user interaction to the next visual update; target < 200 milliseconds |
| CLS (Cumulative Layout Shift) | Total unexpected layout movement during page life; target < 0.1 |
| Perceived Performance | How fast a page feels to the user, independent of actual load time |
| Optimistic UI | Updating the interface immediately before server confirmation, assuming success |
| Critical Rendering Path | The minimum set of resources the browser must process before the first render |
| Resource Hints | Declarative instructions (preload, prefetch, preconnect) that help the browser prioritize resource fetching |
| Code Splitting | Breaking JavaScript bundles into smaller chunks loaded on demand |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Perception trumps reality** — A 3-second load that shows progressive content feels faster than a 2-second load that shows a blank screen; design for perceived speed first
2. **Measure what users feel** — Lab metrics (Lighthouse scores) inform development, but field metrics (Core Web Vitals from real users) reveal actual experience
3. **Budget ruthlessly** — Set performance budgets for JavaScript size, image weight, font count, and third-party scripts; treat every kilobyte as a UX decision
4. **Optimize the critical path** — Identify the minimum resources needed to render meaningful content and prioritize their delivery above everything else
5. **Performance is a shared responsibility** — Design choices (hero images, web fonts, animations), development choices (framework size, third-party scripts), and infrastructure choices (CDN, caching) all contribute; no single team owns performance

### When to Use vs. When to Avoid

**Use when:**
- Every web project benefits from performance-aware UX thinking; it is never inappropriate
- Bounce rates, conversion rates, or engagement metrics are underperforming
- Core Web Vitals scores are failing (red/orange in CrUX data)
- Users are on diverse networks and devices (emerging markets, mobile-heavy audiences)
- Redesigning or rebuilding a product where performance can be designed in from the start

**Avoid over-application when:**
- Micro-optimizing at the cost of readability and maintainability for internal tools with few users
- Removing features that provide genuine user value solely to hit an arbitrary performance budget
- Obsessing over lab scores while ignoring whether real users actually experience problems

---

## Frameworks and Methodologies

### 1. The RAIL Model — user-centric performance goals
1. **Response** — Process user input events within 50ms so interactions feel instant; complex work should yield back to the main thread
2. **Animation** — Produce each frame in under 10ms (targeting 60fps = 16ms per frame minus browser overhead) for smooth visual motion
3. **Idle** — Use idle time to perform deferred work (prefetching, analytics) in 50ms chunks so the main thread stays responsive
4. **Load** — Deliver interactive content within 5 seconds on mid-range mobile over 3G; first meaningful paint should occur within 1-2 seconds

### 2. Performance Optimization Workflow — step-by-step
1. **Audit** — Run Lighthouse, WebPageTest, and check CrUX data for field metrics; identify the worst-performing Core Web Vital
2. **Diagnose** — Use Chrome DevTools Performance panel to trace the bottleneck: is it network, rendering, JavaScript, or images?
3. **Budget** — Set quantitative targets: JavaScript < 200KB compressed, LCP < 2.5s, INP < 200ms, CLS < 0.1
4. **Optimize the critical path** — Inline critical CSS, defer non-essential JavaScript, preload the LCP image, optimize font loading
5. **Implement loading patterns** — Add skeleton screens, lazy load below-fold content, apply blur-up placeholders for images
6. **Validate** — Re-audit and compare field metrics over 28 days to confirm improvements in real user data

---

## How to Apply

### Decision Checklist
- [ ] Is the LCP element identified and prioritized (preloaded, no lazy loading on the LCP image)?
- [ ] Are all images using modern formats (WebP/AVIF) with responsive srcset?
- [ ] Are fonts loaded with `font-display: swap` or `optional` and critical fonts preloaded?
- [ ] Is JavaScript code-split so only the critical path bundle loads initially?
- [ ] Are third-party scripts loaded asynchronously or deferred?
- [ ] Do layout elements have explicit width and height to prevent CLS?
- [ ] Are animations using CSS transforms/opacity only (not layout-triggering properties)?
- [ ] Is `prefers-reduced-motion` respected for all animations?
- [ ] Are skeleton screens or loading states shown for any content that takes > 200ms to appear?
- [ ] Is there a performance budget documented and enforced in CI?

### Implementation Patterns

**Optimistic UI Update:**
```html
<button id="like-btn" class="like-btn" aria-pressed="false">
  <span class="like-icon">&#9829;</span>
  <span id="like-count">42</span>
</button>

<script>
const btn = document.getElementById('like-btn');
const count = document.getElementById('like-count');

btn.addEventListener('click', async () => {
  const isLiked = btn.getAttribute('aria-pressed') === 'true';
  const newState = !isLiked;
  const currentCount = parseInt(count.textContent, 10);

  // Optimistic update: change UI immediately
  btn.setAttribute('aria-pressed', String(newState));
  btn.classList.toggle('like-btn--active', newState);
  count.textContent = newState ? currentCount + 1 : currentCount - 1;

  try {
    const response = await fetch('/api/like', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ liked: newState }),
    });

    if (!response.ok) throw new Error('Server error');
  } catch (error) {
    // Revert on failure
    btn.setAttribute('aria-pressed', String(isLiked));
    btn.classList.toggle('like-btn--active', isLiked);
    count.textContent = currentCount;
    showToast('Could not update. Please try again.');
  }
});
</script>

<style>
.like-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  border: 1px solid #e2e8f0;
  border-radius: 20px;
  background: #fff;
  cursor: pointer;
  transition: all 0.15s ease;
}

.like-btn--active {
  background: #fef2f2;
  border-color: #fca5a5;
  color: #dc2626;
}

.like-btn--active .like-icon {
  transform: scale(1.2);
}

.like-icon {
  transition: transform 0.2s ease;
}
</style>
```

**Skeleton Screen with CSS-only Shimmer:**
```html
<div class="profile-skeleton" aria-hidden="true">
  <div class="skeleton-avatar"></div>
  <div class="skeleton-lines">
    <div class="skeleton-line" style="width: 60%"></div>
    <div class="skeleton-line" style="width: 80%"></div>
    <div class="skeleton-line" style="width: 45%"></div>
  </div>
</div>

<style>
.profile-skeleton {
  display: flex;
  gap: 16px;
  padding: 20px;
}

.skeleton-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  flex-shrink: 0;
}

.skeleton-lines {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.skeleton-line {
  height: 14px;
  border-radius: 4px;
}

.skeleton-avatar,
.skeleton-line {
  background: linear-gradient(
    90deg,
    #e5e7eb 0%,
    #f3f4f6 40%,
    #e5e7eb 80%
  );
  background-size: 300% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}

@keyframes shimmer {
  0% { background-position: 100% 0; }
  100% { background-position: -100% 0; }
}
</style>
```

**Blur-up Image Placeholder Technique:**
```html
<div class="blur-image-wrapper">
  <!-- Tiny base64 placeholder (< 1KB) shown immediately -->
  <img
    src="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
    alt=""
    class="blur-placeholder"
    aria-hidden="true"
  />
  <!-- Full image lazy loaded -->
  <img
    src="hero.webp"
    srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"
    sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 800px"
    alt="Mountain landscape at sunset"
    class="blur-full-image"
    loading="lazy"
    decoding="async"
    onload="this.classList.add('loaded')"
  />
</div>

<style>
.blur-image-wrapper {
  position: relative;
  overflow: hidden;
  aspect-ratio: 16 / 9;
  background: #f3f4f6;
}

.blur-placeholder {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: blur(20px);
  transform: scale(1.1); /* prevent blurred edges from showing */
}

.blur-full-image {
  position: relative;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 0.4s ease;
}

.blur-full-image.loaded {
  opacity: 1;
}
</style>
```

**Respecting prefers-reduced-motion:**
```css
/* Default: animations enabled */
.card {
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.15);
}

.hero-text {
  animation: fadeSlideUp 0.6s ease-out both;
}

@keyframes fadeSlideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Reduced motion: remove movement, keep state changes */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }

  .card:hover {
    transform: none;
    box-shadow: 0 0 0 2px #2563eb; /* use outline instead of motion */
  }
}

/* In JavaScript: check the preference */
/* const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches; */
```

**Font Loading Strategy:**
```html
<!-- Preload critical font to reduce LCP delay -->
<link
  rel="preload"
  href="/fonts/inter-var-latin.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>

<style>
/* Use font-display to control rendering behavior */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var-latin.woff2') format('woff2');
  font-weight: 100 900;
  font-display: optional; /* prevents FOUT entirely; falls back if not cached */
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC;
}

/* System font stack fallback that closely matches Inter metrics */
body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    'Helvetica Neue', Arial, sans-serif;
  /* Size-adjust if needed to reduce CLS from font swap */
}
</style>
```

**Resource Hints for Instant Navigation:**
```html
<head>
  <!-- Preconnect to critical third-party origins -->
  <link rel="preconnect" href="https://api.example.com" />
  <link rel="preconnect" href="https://fonts.googleapis.com" crossorigin />

  <!-- Preload the LCP image (do NOT lazy-load the LCP element) -->
  <link
    rel="preload"
    as="image"
    href="/images/hero.webp"
    fetchpriority="high"
    type="image/webp"
  />

  <!-- Prefetch likely next pages during idle time -->
  <link rel="prefetch" href="/products" />

  <!-- Speculation rules for instant navigation (Chrome 121+) -->
  <script type="speculationrules">
    {
      "prerender": [
        { "where": { "href_matches": "/products/*" }, "eagerness": "moderate" }
      ]
    }
  </script>
</head>
```

### Common Mistakes
1. **Lazy loading the LCP image** — The `loading="lazy"` attribute on the largest above-the-fold image delays its load, directly increasing LCP. Always use `fetchpriority="high"` and avoid lazy loading for the LCP element
2. **Not reserving space for dynamic content** — Images, ads, and embeds without explicit `width` and `height` or `aspect-ratio` cause layout shifts (CLS). Always declare dimensions
3. **Using `font-display: swap` for non-critical fonts** — Swap causes a visible flash of unstyled text (FOUT) and potential CLS. Use `optional` for body fonts to eliminate layout shift, reserving `swap` only for critical heading fonts
4. **Animating layout properties** — Animating `width`, `height`, `top`, `left`, or `margin` triggers expensive layout recalculations. Use `transform` and `opacity` exclusively for performant animations
5. **Ignoring third-party script impact** — Analytics, chat widgets, and ad scripts often account for 50%+ of total JavaScript. Audit, defer, or replace heavy third-party scripts
6. **Optimizing only for fast connections** — Test on throttled 3G with mid-range devices. A site that feels fast on fiber and a MacBook Pro may be unusable on a budget Android phone over cellular data

### Integration with Other Topics
- **27-responsive-web-design** — Responsive images (`srcset`, `sizes`) are both a responsive design and a performance technique; serving a 2400px image to a 400px viewport wastes bandwidth
- **24-web-typography** — Font loading strategy directly impacts both CLS and LCP; variable fonts reduce total font weight while maintaining typographic richness
- **36-modern-web-patterns** — Skeleton screens, infinite scroll, and lazy loading are UX patterns with deep performance implications; this file covers the performance rationale behind those patterns
- **04-microinteractions** — Animation performance (60fps, transform-only, reduced-motion) is where microinteraction design meets performance engineering

---

## Resources

### Essential Reading
- Osmani, A. (2022). *Learning Patterns*. patterns.dev — Free online book on rendering and performance patterns.
- Grigorik, I. (2013). *High Performance Browser Networking*. O'Reilly — Free at hpbn.co.
- Souders, S. (2007). *High Performance Web Sites*. O'Reilly.
- Wagner, J. (2023). "Optimize Interaction to Next Paint." web.dev.

### Online Resources
- [web.dev/vitals](https://web.dev/vitals/) — Google's Core Web Vitals documentation and optimization guides
- [web.dev/learn/performance](https://web.dev/learn/performance/) — Complete performance learning pathway
- [WebPageTest](https://www.webpagetest.org/) — Real-browser performance testing from multiple locations
- [CrUX Dashboard](https://developer.chrome.com/docs/crux/) — Chrome User Experience Report for field data
- [patterns.dev](https://www.patterns.dev/) — Addy Osmani and Lydia Hallie's rendering and performance patterns

### Practice Exercises
1. **Core Web Vitals audit**: Run Lighthouse and check CrUX data for a production website. Identify the worst Core Web Vital, diagnose the root cause, and implement a fix. Measure the before-and-after delta over two weeks of field data
2. **Perceived performance overhaul**: Take a page that shows a spinner during loading. Replace the spinner with a skeleton screen that exactly matches the loaded layout. Conduct a 5-user perception test — ask users which version feels faster
3. **Image optimization pipeline**: Take a page with 10+ images. Convert all to WebP/AVIF with fallbacks, add responsive `srcset` and `sizes`, implement blur-up placeholders, and measure total image weight reduction and LCP improvement
4. **Animation performance profiling**: Open Chrome DevTools Performance panel, record a page with animations, and identify any frames that exceed 16ms. Refactor layout-triggering animations to use `transform` and `opacity` only, then confirm 60fps in the profiler
