# Adaptive Content and Device-Agnostic Strategy

## Profile

### What It Is
Karen McGrane's framework for creating content that adapts to any device, screen size, or channel without being rewritten for each one. Adaptive content treats content as structured, reusable data rather than blobs of HTML tied to a specific layout, enabling organizations to publish once and deliver everywhere. The approach demands that content be separated from presentation at every level, from CMS architecture to editorial workflow.

### Origin and Evolution
- **2007** — NPR develops the COPE model (Create Once, Publish Everywhere), demonstrating structured content for multi-channel delivery across web, mobile apps, and APIs
- **2010** — Mobile traffic surges past early predictions; organizations discover their desktop-first content does not work on small screens, fueling demand for device-agnostic strategy
- **2012** — Karen McGrane publishes *Content Strategy for Mobile*, arguing that mobile is not a subset of desktop and that content must be structured for any channel
- **2013** — Headless CMS platforms emerge (Contentful founded), separating content storage from frontend rendering
- **2015** — McGrane publishes *Going Responsive*, linking content strategy to responsive design implementation
- **2017** — Structured content becomes standard practice as voice interfaces (Alexa, Google Home) expose content to channels with no visual layout at all
- **2020** — Headless and composable CMS architectures (Contentful, Strapi, Sanity, Storyblok) become mainstream, validating the separation McGrane advocated

### Key Authors and References
- **Karen McGrane** — *Content Strategy for Mobile* (2012)
- **Karen McGrane** — *Going Responsive* (2015)
- **Rachel Lovinger** — "Content Modelling: A Master Skill" (2014, A List Apart)
- **Cleve Gibbon** — "Content Modelling for Responsive Design" (2013)
- **Deane Barker** — *Web Content Management* (2016)
- **Carrie Hane & Mike Atherton** — *Designing Connected Content* (2018)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Structured Content | Content broken into discrete, labeled fields instead of monolithic page blobs |
| Content Model | A schema defining content types, their fields, relationships, and constraints |
| COPE | Create Once, Publish Everywhere — the principle that content should be authored once and delivered to multiple channels |
| Headless CMS | A CMS that stores and delivers content through APIs without dictating frontend presentation |
| Content-First Design | Designing with real structured content before choosing layouts or devices |
| Truncation | A flawed responsive strategy that hides content from mobile users instead of rethinking what they need |
| Content Parity | The principle that all users, regardless of device, deserve access to the same content |
| Personalization | Adapting content delivery based on user context (location, behavior, preferences) without duplicating content |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Your content will go places you never imagined** — Design content for reuse across current and future channels (web, mobile apps, voice, watch, kiosk, API consumers); never assume you know all the surfaces
2. **Structure enables freedom** — The more rigorously you model content into discrete, semantic fields, the more flexibly it can be recombined and presented in any context
3. **Separate content from presentation** — Content should carry no assumptions about layout, font size, or screen width; formatting belongs to the delivery layer, not the content layer
4. **Do not create less content for mobile** — Mobile users are not "lite" users; they deserve the same information, adapted to their context, not truncated or hidden behind "view full site" links
5. **The CMS is the strategy** — Your content management system either enables or prevents adaptive content; choosing and configuring the right CMS architecture is a strategic decision, not a technical afterthought

### When to Use vs. When to Avoid

**Use when:**
- Publishing content to multiple channels (website, native app, email, voice assistant, partner API)
- Redesigning a legacy CMS where content is trapped in WYSIWYG page blobs
- Planning a responsive redesign and need to ensure content parity across breakpoints
- Content must be personalized by locale, user segment, or device capability
- Building a design system that consumes content from an API

**Avoid over-application when:**
- A genuinely single-channel product (e.g., a simple marketing site) where the overhead of content modeling is not justified
- Early-stage prototypes where the content model is unknown and rigid structure would slow exploration
- Highly visual, editorial content (longform multimedia stories) where layout and content are intentionally inseparable
- Teams without CMS resources or technical capacity to implement headless architecture

---

## Frameworks and Methodologies

### 1. Content Modeling — define what you have
1. **Inventory content types** — List every distinct kind of content the organization produces (article, product, recipe, FAQ entry, team member bio, event, help topic)
2. **Define fields per type** — For each content type, identify the discrete chunks: title, summary, body, author, date, category, hero image, related items, CTA
3. **Specify field constraints** — Set character limits, required vs. optional status, allowed formats (plain text, rich text, image, reference to another content type)
4. **Map relationships** — Define how content types connect: an article references an author; a product references a category; an event references a venue
5. **Validate against channels** — For each field, confirm it has a meaningful rendering in every target channel (web, app, email, voice); if a field only makes sense visually, reconsider its structure

### 2. Headless CMS Architecture — build the infrastructure
1. **Choose your CMS** — Evaluate headless platforms (Contentful, Strapi, Sanity, Storyblok) against your content model complexity, team size, localization needs, and API requirements
2. **Define the content schema** — Translate your content model into the CMS's type system with proper field types, validations, and references
3. **Set up preview environments** — Enable editors to see how content renders on each channel before publishing
4. **Build the API consumer layer** — Frontends (web, app) fetch content from the CMS API and apply channel-specific presentation
5. **Establish editorial workflow** — Configure draft, review, and publish states with role-based permissions so editors cannot accidentally push unreviewed content live

### 3. Content-First Responsive Strategy — design from content out
1. **Start with content priority** — Before any wireframe, list the content elements for each page type in priority order (most important first)
2. **Design the small screen first** — The mobile layout forces you to prioritize; if content does not earn its place on a small screen, question whether it belongs at all
3. **Use progressive disclosure, not truncation** — Instead of hiding content, provide summaries with paths to detail; every user should be able to reach every piece of content
4. **Test content at every breakpoint** — Real content, not lorem ipsum, must be present in prototypes at every responsive breakpoint to catch overflow, orphan, and readability issues

---

## How to Apply

### Decision Checklist
- [ ] Have you identified all current and likely future channels where this content must appear?
- [ ] Is each content type modeled as a structured schema with discrete, labeled fields?
- [ ] Does every field have validation rules (max length, required, allowed formats)?
- [ ] Is the content free of embedded HTML presentation (inline styles, layout divs, hard-coded widths)?
- [ ] Can the CMS API deliver content as clean, structured data (JSON) to any consumer?
- [ ] Do mobile users have access to the same content as desktop users (content parity)?
- [ ] Is the content localization-ready (translatable strings separated from layout, date/number formats externalized)?
- [ ] Have editors been trained on structured authoring rather than WYSIWYG page building?

### Implementation Patterns

**Content model as JSON schema:**
```json
{
  "contentType": "article",
  "fields": {
    "title": {
      "type": "string",
      "required": true,
      "maxLength": 100,
      "description": "The headline displayed on all channels"
    },
    "slug": {
      "type": "string",
      "required": true,
      "pattern": "^[a-z0-9]+(-[a-z0-9]+)*$",
      "description": "URL-safe identifier, auto-generated from title"
    },
    "summary": {
      "type": "string",
      "required": true,
      "maxLength": 200,
      "description": "Used in cards, search results, social shares, and voice read-aloud"
    },
    "body": {
      "type": "richText",
      "required": true,
      "description": "Structured rich text, not raw HTML"
    },
    "author": {
      "type": "reference",
      "referenceType": "author",
      "required": true
    },
    "category": {
      "type": "reference",
      "referenceType": "category",
      "required": true
    },
    "heroImage": {
      "type": "image",
      "required": false,
      "fields": {
        "alt": { "type": "string", "required": true },
        "caption": { "type": "string", "required": false }
      }
    },
    "publishedAt": {
      "type": "datetime",
      "required": true
    },
    "relatedArticles": {
      "type": "array",
      "items": { "type": "reference", "referenceType": "article" },
      "maxItems": 3
    }
  }
}
```

**Fetching structured content from a headless CMS API:**
```javascript
// Fetch an article and render it on the web channel
async function getArticle(slug) {
  const response = await fetch(
    `https://api.cms.example.com/articles?slug=${slug}&include=author,category`
  );
  const data = await response.json();
  return data;
}

// The same API powers mobile, email, and voice channels.
// Each consumer applies its own presentation layer.
// The content never contains <div> or inline styles.
```

**Responsive content with progressive disclosure instead of truncation:**
```html
<!-- ANTI-PATTERN: truncating content on mobile -->
<p class="description desktop-only">
  Full description visible only on desktop...
</p>
<p class="description mobile-only">
  Shortened version for mo...
</p>

<!-- BETTER: progressive disclosure available to all users -->
<div class="description">
  <p>Short summary visible at all breakpoints.</p>
  <details>
    <summary>Read more</summary>
    <p>Extended description available to every user who wants it,
       regardless of device. No content is hidden or removed.</p>
  </details>
</div>
```

```css
/* Progressive disclosure styling that adapts to viewport */
.description details {
  margin-top: 0.5rem;
}

/* On larger screens, expand by default since space allows */
@media (min-width: 768px) {
  .description details[open] summary {
    display: none;
  }
  .description details {
    /* Auto-open on desktop via JS, or use open attribute */
  }
}

/* On smaller screens, keep collapsed to prioritize space */
@media (max-width: 767px) {
  .description details summary {
    color: var(--color-link);
    cursor: pointer;
    font-weight: 600;
  }
}
```

**Multi-channel content delivery from a single source:**
```javascript
// Same API, different presentation per channel
async function renderArticle(article, context) {
  // Voice channel: only title and summary, no markup
  if (context.channel === 'voice') {
    return { title: article.title, text: article.summary };
  }

  // Email channel: summary + link, no heavy media
  if (context.channel === 'email') {
    return {
      title: article.title,
      preview: article.summary,
      link: `https://example.com/articles/${article.slug}`,
    };
  }

  // Web channel: full content with images and related articles
  return {
    title: article.title,
    summary: article.summary,
    body: article.body,
    heroImage: article.heroImage,
    author: article.author,
    relatedArticles: article.relatedArticles,
  };
}
```

**Localization-ready content fields:**
```json
{
  "name": {
    "en-US": "Wireless Headphones",
    "it-IT": "Cuffie Wireless",
    "de-DE": "Kabellose Kopfhoerer"
  },
  "price": { "amount": 299.00, "currency": "USD" }
}
```

### Common Mistakes
1. **Treating mobile as a lesser channel** — Hiding content behind "view desktop site" or truncating descriptions to fewer characters; mobile users have the same needs, just a different context
2. **WYSIWYG blob content** — Storing content as a single rich-text field with embedded layout HTML, making it impossible to reuse on any other channel or redesign the site without re-entering content
3. **Over-modeling too early** — Creating an elaborate 40-field content model before understanding actual editorial needs; start with the minimum viable model and iterate based on real authoring friction
4. **Ignoring the editorial experience** — Building a perfect API-driven architecture but leaving editors with an unusable CMS interface; adaptive content only works if authors can actually create structured content efficiently
5. **Confusing responsive images with responsive content** — Using srcset for images is only one piece; the text, the data structure, and the interaction patterns must also adapt to context

### Integration with Other Topics
- **27-responsive-web-design** — Responsive design handles presentation adaptation; adaptive content handles content adaptation; they are complementary, not interchangeable
- **30-content-strategy** — Adaptive content is the structural implementation of content strategy; without the substance, workflow, and governance that Halvorson defines, structured content has no direction
- **17-information-architecture** — Content models define the atomic units; information architecture defines how those units are grouped, labeled, and navigated across the site

---

## Resources

### Essential Reading
- McGrane, K. (2012). *Content Strategy for Mobile*. A Book Apart.
- McGrane, K. (2015). *Going Responsive*. A Book Apart.
- Hane, C. & Atherton, M. (2018). *Designing Connected Content*. New Riders.
- Barker, D. (2016). *Web Content Management: Systems, Features, and Best Practices*. O'Reilly.

### Online Resources
- [Karen McGrane's website](https://karenmcgrane.com/) — Articles and talks on adaptive content and responsive content strategy
- [NPR COPE Model Case Study](https://www.programmableweb.com/news/cope-create-once-publish-everywhere/2009/10/13) — The original multi-channel content architecture that proved COPE at scale
- [Contentful: Content Modeling Guide](https://www.contentful.com/help/content-modelling-basics/) — Practical guide to building structured content in a headless CMS
- [A List Apart: Content Modelling](https://alistapart.com/article/content-modelling-a-master-skill/) — Rachel Lovinger's foundational article on content modeling as a UX skill
- [Sanity.io: Structured Content](https://www.sanity.io/structured-content) — Resources on implementing structured, portable content

### Practice Exercises
1. **Content model design**: Choose a familiar content type (recipe, job listing, event). Define its structured model with at least eight fields, specifying type, constraints, and which channels each field serves (web, mobile, email, voice)
2. **Channel adaptation**: Take a single article from any website. Rewrite its content as structured JSON fields (title, summary, body, author, image with alt text). Then write out how each field would render differently on web, email, and a voice assistant
3. **CMS evaluation**: Sign up for free tiers of two headless CMS platforms (e.g., Contentful and Sanity). Implement the same content model in both. Compare the editorial experience, API output, and localization support
4. **Truncation audit**: Browse a responsive website at both desktop and mobile widths. Identify every instance where content is truncated, hidden, or removed on the smaller viewport. For each, propose an alternative using progressive disclosure or content restructuring
