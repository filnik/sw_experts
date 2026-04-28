# Modern Web UI Patterns

## Profile

### What It Is
A comprehensive catalog of recurring UI patterns that have emerged as standard solutions to common interaction problems on the modern web. These patterns — spanning navigation, loading states, scrolling, modals, lists, and search — represent collective design wisdom distilled from millions of websites and validated through extensive user research. Mastering these patterns allows designers and developers to make informed decisions about which solution fits a given context rather than reinventing interactions from scratch.

### Origin and Evolution
- **2006–2010** — Early pattern libraries emerge (Yahoo Design Pattern Library, Welie.com); the web matures beyond brochure sites into interactive applications
- **2011–2013** — Mobile-first thinking introduces bottom navigation, hamburger menus, and pull-to-refresh; Luke Wroblewski publishes *Mobile First*
- **2014–2016** — Skeleton screens popularized by Facebook and LinkedIn; infinite scroll becomes ubiquitous in social feeds
- **2017–2019** — Command palettes gain traction (Slack, VS Code, Superhuman); virtual scrolling matures for large datasets
- **2020–2024** — Bottom sheets become standard on mobile; mega menus evolve with accessibility focus; Smashing Magazine publishes its comprehensive design patterns series by Vitaly Friedman
- **2025–present** — AI-augmented search patterns, adaptive loading based on device capabilities, and command-palette-first interfaces become mainstream

### Key Authors and References
- **Vitaly Friedman** — *Smart Interface Design Patterns* (Smashing Magazine, 2020–2024)
- **Luke Wroblewski** — *Mobile First* (2011) and *Web Form Design* (2008)
- **Jenifer Tidwell** — *Designing Interfaces: Patterns for Effective Interaction Design* (3rd ed., 2020)
- **Brad Frost** — *Atomic Design* (2016) — component composition underpinning pattern implementation
- **Adam Silver** — *Form Design Patterns* (Smashing Magazine, 2018)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Navigation Pattern | A reusable approach to helping users move through an information architecture |
| Skeleton Screen | A placeholder UI that mimics the page layout before content loads |
| Infinite Scroll | Continuously loading content as the user scrolls, removing traditional pagination |
| Virtual Scrolling | Rendering only the visible portion of a long list, recycling DOM nodes as the user scrolls |
| Command Palette | A keyboard-driven search interface for navigating actions and content (Cmd+K) |
| Bottom Sheet | A mobile UI surface that slides up from the bottom of the screen for contextual actions |
| Faceted Search | Filtering search results by multiple independent dimensions simultaneously |
| Optimistic UI | Updating the interface immediately before server confirmation, assuming success |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Context determines the pattern** — No pattern is universally correct; the right choice depends on content type, user intent, device, and frequency of use
2. **Reduce interaction cost** — Every pattern should minimize the physical and cognitive effort required to accomplish a task
3. **Progressive disclosure over information overload** — Show users what they need now and provide clear paths to deeper content
4. **Familiar conventions over novel inventions** — Users bring expectations from other sites; deviate from conventions only when the usability gain is substantial
5. **Graceful degradation and enhancement** — Patterns must work across devices and connection speeds, starting from a functional baseline and layering richness

### When to Use vs. When to Avoid

**Use when:**
- Building production web applications that serve diverse user populations
- Designing information-heavy interfaces (dashboards, e-commerce, content platforms)
- Standardizing interaction behavior across a design system
- Evaluating existing UI for consistency and usability improvements
- Making architectural decisions about content loading and navigation strategy

**Avoid over-application when:**
- Blindly copying patterns without understanding the user context (e.g., infinite scroll for legal documents)
- The pattern adds complexity without solving a real user problem
- A simpler, less "trendy" approach would serve users better
- Accessibility requirements conflict with the pattern and no accessible alternative exists

---

## Frameworks and Methodologies

### 1. Pattern Selection Framework — step-by-step
1. **Identify the user task** — What is the user trying to accomplish? (browse, search, compare, act)
2. **Characterize the content** — How much content exists? How frequently does it update? Is it structured or freeform?
3. **Assess the context** — What devices will be used? What is the user's expertise level? What are the performance constraints?
4. **Survey candidate patterns** — List 2-3 patterns that could solve the interaction problem
5. **Evaluate trade-offs** — Score each pattern on discoverability, accessibility, performance, and implementation cost
6. **Prototype and test** — Build a lightweight version and validate with real users before committing

### 2. Navigation Architecture Decision Tree — step-by-step
1. Count primary navigation items: if 2-5, use tab bar or horizontal navigation; if 6+, use mega menu or sidebar
2. Determine mobile strategy: if 5 or fewer items, use bottom navigation; if more, use hamburger with prioritized items visible
3. Assess depth: if 3+ levels deep, add breadcrumbs for wayfinding
4. Evaluate frequency of cross-section jumping: if high, consider a command palette as a complement
5. Test navigation with the "trunk test" — can users identify where they are, where they can go, and how to get back?

---

## How to Apply

### Decision Checklist
- [ ] Have I identified the primary user task this pattern serves?
- [ ] Does this pattern work on mobile, tablet, and desktop?
- [ ] Is the pattern accessible via keyboard and screen reader?
- [ ] Have I considered the performance implications (DOM size, network requests, JavaScript payload)?
- [ ] Does the pattern degrade gracefully without JavaScript?
- [ ] Have I tested the pattern with real content (not just placeholder data)?
- [ ] Does the loading state communicate progress and prevent layout shift?
- [ ] Is there a clear fallback for error states?

### Implementation Patterns

**Skeleton Screen — CSS and HTML:**
```html
<!-- Skeleton placeholder that mirrors the final card layout -->
<div class="card card--skeleton" aria-hidden="true" aria-label="Loading content">
  <div class="skeleton-line skeleton-image"></div>
  <div class="skeleton-line skeleton-title"></div>
  <div class="skeleton-line skeleton-text"></div>
  <div class="skeleton-line skeleton-text skeleton-text--short"></div>
</div>

<style>
.card--skeleton {
  padding: 16px;
  border-radius: 8px;
  background: #fff;
}

.skeleton-line {
  background: linear-gradient(
    90deg,
    #e2e8f0 25%,
    #f1f5f9 50%,
    #e2e8f0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
  border-radius: 4px;
  margin-bottom: 12px;
}

.skeleton-image {
  width: 100%;
  height: 180px;
}

.skeleton-title {
  width: 70%;
  height: 20px;
}

.skeleton-text {
  width: 100%;
  height: 14px;
}

.skeleton-text--short {
  width: 40%;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
</style>
```

**Infinite Scroll with Intersection Observer:**
```html
<ul id="feed" class="feed" role="feed" aria-busy="false">
  <!-- Items rendered here -->
</ul>
<div id="sentinel" class="sentinel" aria-hidden="true"></div>
<p id="feed-status" class="sr-only" role="status" aria-live="polite"></p>

<script>
class InfiniteScroll {
  constructor(feedEl, sentinelEl, statusEl) {
    this.feed = feedEl;
    this.sentinel = sentinelEl;
    this.status = statusEl;
    this.page = 1;
    this.loading = false;
    this.hasMore = true;

    this.observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !this.loading && this.hasMore) {
          this.loadMore();
        }
      },
      { rootMargin: '200px' }
    );
    this.observer.observe(this.sentinel);
  }

  async loadMore() {
    this.loading = true;
    this.feed.setAttribute('aria-busy', 'true');
    this.status.textContent = 'Loading more items...';

    try {
      const response = await fetch(`/api/items?page=${this.page}`);
      const data = await response.json();

      data.items.forEach(item => {
        const li = document.createElement('li');
        li.className = 'feed-item';
        li.setAttribute('tabindex', '0');
        li.innerHTML = `<h3>${item.title}</h3><p>${item.summary}</p>`;
        this.feed.appendChild(li);
      });

      this.page++;
      this.hasMore = data.hasMore;
      this.status.textContent = `Loaded ${data.items.length} more items.`;
    } catch (error) {
      this.status.textContent = 'Failed to load items. Scroll to retry.';
    } finally {
      this.loading = false;
      this.feed.setAttribute('aria-busy', 'false');
    }
  }
}

new InfiniteScroll(
  document.getElementById('feed'),
  document.getElementById('sentinel'),
  document.getElementById('feed-status')
);
</script>
```

**Command Palette (Cmd+K):**
```html
<dialog id="command-palette" class="command-palette" aria-label="Command palette">
  <div class="command-palette__inner">
    <input
      type="search"
      id="command-input"
      class="command-palette__input"
      placeholder="Type a command or search..."
      autocomplete="off"
      role="combobox"
      aria-expanded="true"
      aria-controls="command-results"
      aria-activedescendant=""
    />
    <ul id="command-results" class="command-palette__results" role="listbox">
      <!-- Results rendered dynamically -->
    </ul>
  </div>
</dialog>

<style>
.command-palette {
  border: none;
  border-radius: 12px;
  padding: 0;
  width: min(640px, 90vw);
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
  top: 20vh;
}

.command-palette::backdrop {
  background: rgba(0, 0, 0, 0.5);
}

.command-palette__input {
  width: 100%;
  padding: 16px 20px;
  font-size: 18px;
  border: none;
  border-bottom: 1px solid #e2e8f0;
  outline: none;
}

.command-palette__results {
  max-height: 320px;
  overflow-y: auto;
  list-style: none;
  margin: 0;
  padding: 8px;
}

.command-palette__results li {
  padding: 10px 16px;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
}

.command-palette__results li[aria-selected="true"],
.command-palette__results li:hover {
  background-color: #f1f5f9;
}
</style>

<script>
const palette = document.getElementById('command-palette');
const input = document.getElementById('command-input');
const results = document.getElementById('command-results');
let activeIndex = -1;

const commands = [
  { label: 'Go to Dashboard', action: () => location.href = '/dashboard', section: 'Navigation' },
  { label: 'Go to Settings', action: () => location.href = '/settings', section: 'Navigation' },
  { label: 'Create New Project', action: () => location.href = '/new', section: 'Actions' },
  { label: 'Toggle Dark Mode', action: () => document.body.classList.toggle('dark'), section: 'Actions' },
];

document.addEventListener('keydown', (e) => {
  if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
    e.preventDefault();
    palette.showModal();
    input.value = '';
    input.focus();
    renderResults(commands);
  }
});

palette.addEventListener('click', (e) => {
  if (e.target === palette) palette.close();
});

input.addEventListener('input', () => {
  const query = input.value.toLowerCase();
  const filtered = commands.filter(c => c.label.toLowerCase().includes(query));
  renderResults(filtered);
});

input.addEventListener('keydown', (e) => {
  const items = results.querySelectorAll('li');
  if (e.key === 'ArrowDown') {
    e.preventDefault();
    activeIndex = Math.min(activeIndex + 1, items.length - 1);
    updateSelection(items);
  } else if (e.key === 'ArrowUp') {
    e.preventDefault();
    activeIndex = Math.max(activeIndex - 1, 0);
    updateSelection(items);
  } else if (e.key === 'Enter' && activeIndex >= 0) {
    e.preventDefault();
    items[activeIndex].click();
  } else if (e.key === 'Escape') {
    palette.close();
  }
});

function renderResults(items) {
  activeIndex = -1;
  results.innerHTML = items.map((item, i) =>
    `<li role="option" id="cmd-${i}" data-index="${i}">${item.label}</li>`
  ).join('');

  results.querySelectorAll('li').forEach((li, i) => {
    li.addEventListener('click', () => {
      items[i].action();
      palette.close();
    });
  });
}

function updateSelection(items) {
  items.forEach((item, i) => {
    item.setAttribute('aria-selected', i === activeIndex ? 'true' : 'false');
  });
  if (activeIndex >= 0) {
    input.setAttribute('aria-activedescendant', `cmd-${activeIndex}`);
  }
}
</script>
```

### Common Mistakes
1. **Using infinite scroll for goal-oriented tasks** — Infinite scroll works for browsing feeds but frustrates users who need to find a specific item, compare items, or reach the footer. Use pagination or "Load more" for catalogs and search results
2. **Hamburger menu hiding critical navigation** — Concealing all navigation behind a hamburger icon reduces discoverability by 21% (NNg research). Keep primary actions visible; use hamburger only for secondary items
3. **Skeleton screens that do not match the real layout** — If the skeleton shape differs significantly from the loaded content, it increases perceived loading time rather than reducing it. Match dimensions precisely
4. **Modal dialogs for non-blocking content** — Using modals to display information that does not require immediate action interrupts the user flow. Use inline expansion, toasts, or slide-over panels instead
5. **Command palette as the only navigation method** — A command palette is a power-user accelerator, not a replacement for visible navigation. Always provide a discoverable primary navigation alongside it
6. **Infinite scroll without keyboard and screen reader support** — Many infinite scroll implementations break focus management and fail to announce new content. Always use `role="feed"`, `aria-busy`, and `aria-live` regions

### Integration with Other Topics
- **17-information-architecture** — Navigation patterns are the visible expression of the underlying IA; poor IA cannot be fixed by better navigation UI
- **06-web-usability** — Nielsen's heuristics (visibility of system status, user control) directly inform pattern selection: skeleton screens address visibility, "Load more" preserves user control
- **27-responsive-web-design** — Pattern behavior must adapt across breakpoints: bottom navigation on mobile, horizontal tabs on desktop, mega menus on wide screens
- **37-web-performance-ux** — Loading patterns (skeleton screens, lazy loading, virtual scroll) are inseparable from performance strategy; a fast site with no loading states still feels broken during network delays

---

## Resources

### Essential Reading
- Friedman, V. (2020–2024). *Smart Interface Design Patterns*. Smashing Magazine.
- Tidwell, J., Brewer, C., & Valencia, A. (2020). *Designing Interfaces*, 3rd Edition. O'Reilly.
- Wroblewski, L. (2011). *Mobile First*. A Book Apart.
- Silver, A. (2018). *Form Design Patterns*. Smashing Magazine.

### Online Resources
- [Smashing Magazine Design Patterns](https://www.smashingmagazine.com/category/design-patterns) — Vitaly Friedman's comprehensive pattern articles
- [UI Patterns](https://ui-patterns.com/) — Catalog of user interface design patterns with examples
- [Mobbin](https://mobbin.com/) — Curated library of real-world mobile and web UI patterns
- [MDN: Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) — Foundation for lazy loading and infinite scroll
- [WAI-ARIA Feed Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/feed/) — Accessible infinite scroll specification

### Practice Exercises
1. **Navigation audit**: Take a complex web application (e.g., an e-commerce site). Document every navigation mechanism. Identify which pattern each uses and whether it is the best fit for the content depth and user task
2. **Skeleton screen implementation**: Build a card-based layout that shows skeleton screens during loading. Ensure the skeleton dimensions exactly match the loaded content to prevent layout shift (CLS < 0.1)
3. **Pattern comparison test**: Implement the same product listing three ways — infinite scroll, pagination, and "Load more." Conduct a usability test with 5 users asking them to find a specific item and reach the footer. Document which pattern users prefer and why
4. **Command palette build**: Implement a command palette for an existing project. Include navigation commands, action commands, and fuzzy search. Measure whether power users adopt it and whether it reduces time-to-action
