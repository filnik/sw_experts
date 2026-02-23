# Web Usability

## Profile

### What It Is
Steve Krug's approach to web usability is grounded in the principle that web pages should be self-evident — users should be able to "get it" without expending any effort thinking about it. His work emphasizes that users do not read web pages; they scan them, they do not make optimal choices; they satisfice, and they do not figure things out; they muddle through. This practical philosophy has shaped how millions of websites are designed and tested.

### Origin and Evolution
- **2000** — Steve Krug publishes *Don't Make Me Think*, immediately becoming the most recommended usability book for web practitioners
- **2006** — Second edition adds chapters on accessibility and mobile web usability
- **2009** — *Rocket Surgery Made Easy* introduces the monthly guerrilla usability testing method
- **2014** — Third edition of *Don't Make Me Think* (subtitled *Revisited*) updates for mobile-first, responsive design, and modern web patterns
- **2015-present** — Krug's principles continue to influence component libraries, design systems, and UX education

### Key Authors and References
- **Steve Krug** — *Don't Make Me Think, Revisited* (3rd edition, 2014)
- **Steve Krug** — *Rocket Surgery Made Easy* (2009)
- **Jakob Nielsen** — *Designing Web Usability* (1999)
- **Gerry McGovern** — *Top Tasks: How to Focus on What Matters* (2018)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Satisficing | Users choose the first reasonable option, not the best option |
| Scanning | Users scan pages in F-patterns rather than reading word by word |
| Muddling Through | Users do not learn how a site works; they improvise each time |
| Self-Evident | The ideal page requires zero thought to understand and navigate |
| Trunk Test | A quick check to verify any page communicates where you are and how to navigate |
| Goodwill Reservoir | Each user starts with a finite amount of patience that poor design depletes |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Do not make me think** — Every element should be self-evident or at worst self-explanatory; if users have to think about whether something is clickable, where to find a feature, or what a label means, the design has failed
2. **People do not read, they scan** — Design for scanning with clear visual hierarchy, short paragraphs, bulleted lists, and meaningful headings
3. **People satisfice** — Users choose the first option that looks like it might work rather than evaluating all options; make the right path the most obvious one
4. **People muddle through** — Users rarely learn how a site works; they find something that works and stick with it even if it is not the intended path
5. **Eliminate unnecessary words** — Remove half the words, then remove half of what is left; every word on the page competes for attention

### When to Use vs. When to Avoid

**Use when:**
- Designing or evaluating any consumer-facing website or web application
- Building navigation structures, information architecture, and page layouts
- Planning low-budget usability testing with small teams
- Training developers and stakeholders who are new to usability concepts
- Redesigning existing sites where users report confusion or high bounce rates

**Avoid over-application when:**
- The application is a specialized professional tool where users invest time learning (e.g., CAD, video editing)
- You are designing for a deeply technical audience who expects information density
- The problem requires quantitative research (Krug's methods are primarily qualitative)
- You need statistically significant data to make business decisions (use A/B testing instead)

---

## Frameworks and Methodologies

### 1. The Trunk Test — step-by-step
Imagine being blindfolded, driven around, and dropped on a random page of the website. Can you answer these questions?

1. **What site is this?** — Is the site identity (logo, name) visible and consistent?
2. **What page am I on?** — Is the page title or heading clear?
3. **What are the major sections?** — Is the primary navigation visible and understandable?
4. **What are my options at this level?** — Is the local navigation or secondary navigation clear?
5. **Where am I in the scheme of things?** — Are breadcrumbs or "you are here" indicators present?
6. **How can I search?** — Is the search box easy to find?

### 2. Krug's Monthly Usability Testing — step-by-step
1. **Schedule one morning per month** — Block 2-3 hours; consistency matters more than intensity
2. **Recruit 3 users** — Not 8, not 12; three users per session reveal most critical issues
3. **Write 3-5 key tasks** — Focus on what matters most (signup, purchase, finding information)
4. **Run 20-minute sessions** — Think-aloud protocol; observer takes notes, facilitator guides
5. **Debrief over lunch** — The whole team watches; discuss and identify the top 3 problems
6. **Fix the top problem** — Do not create a backlog of 50 issues; fix the worst one before next month
7. **Repeat** — Monthly cadence creates continuous improvement without analysis paralysis

### 3. Self-Evident vs Self-Explanatory Scale
1. **Self-evident** (ideal) — The user understands instantly without any thought
2. **Self-explanatory** (acceptable) — The user needs a moment but figures it out from visual cues
3. **Requires explanation** (problematic) — The user needs external help, a tooltip, or documentation
4. **Incomprehensible** (failure) — The user cannot figure it out at all

---

## How to Apply

### Decision Checklist
- [ ] Can a first-time visitor identify the site's purpose within 5 seconds?
- [ ] Is the navigation structured with no more than 5-7 top-level items?
- [ ] Does every page pass the trunk test?
- [ ] Are clickable elements visually distinct from non-clickable elements?
- [ ] Is the search box visible on every page without scrolling?
- [ ] Are forms as short as possible with clear labels above fields?
- [ ] Have unnecessary words been removed from all headings and body text?
- [ ] Can a user complete the primary task in 3 clicks or fewer from the homepage?
- [ ] Is the page scannable (headings, bullet points, short paragraphs)?
- [ ] Has the design been tested with at least 3 real users?

### Implementation Patterns

**Clear, Scannable Navigation:**
```html
<!-- Good: Clear labels, visible hierarchy, current page indicator -->
<nav class="main-nav" aria-label="Main navigation">
  <a href="/" class="nav-logo">
    <img src="/logo.svg" alt="ShopName" width="120" height="32">
  </a>
  <ul class="nav-links">
    <li><a href="/products" aria-current="page" class="active">Products</a></li>
    <li><a href="/pricing">Pricing</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/support">Support</a></li>
  </ul>
  <form class="nav-search" role="search" action="/search">
    <label for="search" class="sr-only">Search</label>
    <input type="search" id="search" name="q" placeholder="Search...">
    <button type="submit" aria-label="Search"><svg>...</svg></button>
  </form>
</nav>

<style>
.main-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 2rem;
  border-bottom: 1px solid #e2e8f0;
  background: white;
  position: sticky;
  top: 0;
  z-index: 100;
}
.nav-links {
  display: flex;
  gap: 0.25rem;
  list-style: none;
  margin: 0;
  padding: 0;
}
.nav-links a {
  padding: 0.5rem 1rem;
  text-decoration: none;
  color: #475569;
  border-radius: 0.375rem;
  font-weight: 500;
  transition: background 0.15s;
}
.nav-links a:hover { background: #f1f5f9; color: #0f172a; }
.nav-links a.active {
  background: #eff6ff;
  color: #1d4ed8;
  font-weight: 600;
}
</style>
```

**Breadcrumbs for "You Are Here" Orientation:**
```html
<!-- Breadcrumbs pass the trunk test: users always know where they are -->
<nav class="breadcrumbs" aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/electronics">Electronics</a></li>
    <li><a href="/electronics/headphones">Headphones</a></li>
    <li aria-current="page">Sony WH-1000XM5</li>
  </ol>
</nav>

<style>
.breadcrumbs ol {
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  padding: 0;
  margin: 0;
  font-size: 0.875rem;
  color: #64748b;
}
.breadcrumbs li + li::before {
  content: "/";
  margin: 0 0.5rem;
  color: #cbd5e1;
}
.breadcrumbs a {
  color: #3b82f6;
  text-decoration: none;
}
.breadcrumbs a:hover { text-decoration: underline; }
.breadcrumbs [aria-current="page"] {
  color: #0f172a;
  font-weight: 500;
}
</style>
```

**Good vs Bad: Call to Action Clarity:**
```html
<!-- BAD: Unclear, too many competing options -->
<div class="bad-cta-section">
  <button class="btn">Submit</button>
  <button class="btn">Learn More</button>
  <button class="btn">Contact Us</button>
  <button class="btn">Download</button>
  <button class="btn">Subscribe</button>
</div>

<!-- GOOD: Clear primary action, secondary is visually subordinate -->
<div class="good-cta-section">
  <button class="btn-primary">Start Free Trial</button>
  <a href="/demo" class="btn-text">or watch a 2-min demo</a>
</div>

<style>
/* Bad: all buttons look the same — user has to think */
.bad-cta-section .btn {
  background: #3b82f6;
  color: white;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 0.375rem;
  margin: 0.25rem;
}

/* Good: clear visual hierarchy */
.good-cta-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}
.btn-primary {
  background: #2563eb;
  color: white;
  padding: 1rem 2rem;
  border: none;
  border-radius: 0.5rem;
  font-size: 1.125rem;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(37,99,235,0.3);
}
.btn-text {
  color: #64748b;
  text-decoration: none;
  font-size: 0.875rem;
}
.btn-text:hover { text-decoration: underline; }
</style>
```

**Eliminating Words — Before and After:**
```html
<!-- BEFORE: 47 words -->
<div class="hero-bad">
  <h1>Welcome to Our Amazing Online Platform for Finding and Purchasing Products</h1>
  <p>We are dedicated to providing you with the most comprehensive and extensive
  selection of high-quality products available anywhere on the internet at
  competitive and affordable prices.</p>
</div>

<!-- AFTER: 14 words -->
<div class="hero-good">
  <h1>Quality products, honest prices</h1>
  <p>Search 50,000+ items. Free shipping over $35.</p>
</div>
```

### Common Mistakes
1. **Designing for yourself** — Developers and designers are not typical users; what seems obvious to you may confuse someone encountering the site for the first time
2. **Religious debates over design choices** — Krug warns against endless arguments about whether users prefer dropdowns or mega menus; test with 3 users and let data decide
3. **Testing too late** — Waiting until the product is finished means expensive changes; test early with paper prototypes
4. **Asking users what they want** — Users are bad at predicting their own behavior; observe what they do, not what they say
5. **Fixing everything at once** — Krug advocates fixing the single worst problem after each test session rather than creating an overwhelming backlog

### Integration with Other Topics
- **[Usability Heuristics (05)](../usability-research/05-usability-heuristics.md)** — Krug's principles operationalize several of Nielsen's heuristics, particularly visibility of system status and recognition over recall
- **[Information Architecture (17)](../information-architecture/17-information-architecture.md)** — The trunk test directly evaluates IA quality; navigation and labeling are core concerns of both disciplines
- **[UX Research Methods (07)](../usability-research/07-ux-research-methods.md)** — Krug's guerrilla testing method is one approach within the broader evaluative research toolkit

---

## Resources

### Essential Reading
- Krug, S. (2014). *Don't Make Me Think, Revisited*. New Riders. 3rd edition.
- Krug, S. (2009). *Rocket Surgery Made Easy*. New Riders.
- McGovern, G. (2018). *Top Tasks: How to Focus on What Matters*. Silver Beach.

### Online Resources
- [Steve Krug's Website — Advanced Common Sense](http://www.sensible.com/)
- [Krug's Usability Test Script (free download)](http://www.sensible.com/rocket-surgery-made-easy/)
- [Nielsen Norman Group — How Users Read on the Web](https://www.nngroup.com/articles/how-users-read-on-the-web/)
- [Nielsen Norman Group — F-Shaped Pattern of Reading](https://www.nngroup.com/articles/f-shaped-pattern-reading-web-content/)

### Practice Exercises
1. **Trunk test audit** — Print 5 random interior pages from your site; hand them to a colleague who has never seen the site and ask the 6 trunk test questions; note every hesitation
2. **Word-halving exercise** — Take any page of marketing copy and cut the word count by 50% without losing meaning; then do it again; note how the message improves
3. **Run a Krug-style test** — Recruit 3 colleagues from outside your team, write 3 tasks, record 20-minute sessions, debrief, and fix the single worst problem found
4. **Scan test** — Show a page to someone for 5 seconds, then hide it and ask what they remember; this reveals whether your visual hierarchy communicates the right things
