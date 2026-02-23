# Psychology for Designers

## Profile

### What It Is
Psychology for designers applies cognitive science and behavioral psychology to user interface design, translating findings from decades of psychological research into actionable design patterns. This knowledge base draws from Susan Weinschenk's practical catalog of psychological principles and Jeff Johnson's systematic framework for understanding how human cognition shapes interface expectations. Together they provide the scientific foundation for why certain design patterns work and others fail.

### Origin and Evolution
- **1956** — George Miller publishes "The Magical Number Seven, Plus or Minus Two," establishing memory capacity limits that still influence UI design
- **1983** — Don Norman and Andrew Ortony formalize the concept of mental models in human-computer interaction
- **1988** — Don Norman's *The Design of Everyday Things* bridges cognitive psychology and product design
- **2010** — Jeff Johnson publishes *Designing with the Mind in Mind*, systematizing perception and cognition for UI designers
- **2011** — Susan Weinschenk publishes *100 Things Every Designer Needs to Know About People*, making psychology accessible to practitioners
- **2014** — Second editions of both books incorporate mobile, touch, and responsive design contexts
- **2020** — Third edition of Johnson's book adds chapters on learning from experience and attention

### Key Authors and References
- **Susan Weinschenk** — *100 Things Every Designer Needs to Know About People* (2011, 2nd edition 2020)
- **Jeff Johnson** — *Designing with the Mind in Mind* (2010, 3rd edition 2020)
- **Daniel Kahneman** — *Thinking, Fast and Slow* (2011)
- **Don Norman** — *The Design of Everyday Things* (revised edition, 2013)
- **B.J. Fogg** — *Persuasive Technology* (2003)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Working Memory Limits | Humans can hold approximately 4 (not 7) chunks of information in active memory at once |
| Mental Models | Internal representations of how a system works, built from prior experience |
| Change Blindness | Users fail to notice changes in visual scenes if the change is not in their focus of attention |
| Dual Process Theory | System 1 (fast, intuitive) vs System 2 (slow, deliberate) thinking drives user decisions |
| Social Proof | People look to others' behavior to determine correct action in uncertain situations |
| Cognitive Tunneling | Under stress, attention narrows and users miss peripheral information |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Design for the brain you have, not the brain you wish you had** — Human cognition has fixed constraints (attention, memory, perception); fighting these constraints produces poor interfaces
2. **People see what they expect to see** — Perception is driven by mental models and expectations as much as by what is actually on screen; break conventions carefully
3. **The less users have to think, the more they accomplish** — Minimize cognitive load by leveraging recognition, chunking, and progressive disclosure
4. **Emotion drives decisions more than logic** — Users form emotional impressions in milliseconds; trust, credibility, and aesthetic appeal are processed before content is read
5. **Behavior is shaped by context, not just personality** — The same user behaves differently depending on device, environment, time pressure, and social context

### When to Use vs. When to Avoid

**Use when:**
- Designing interfaces that must work for broad, diverse user populations
- Explaining to stakeholders why a design decision is supported by evidence
- Resolving design disagreements with research-backed principles instead of opinions
- Optimizing conversion flows, onboarding sequences, or error recovery
- Creating design guidelines or pattern libraries that need a rationale layer

**Avoid over-application when:**
- Using psychology to manipulate users into decisions that harm their interests (dark patterns)
- Over-generalizing laboratory findings to all contexts without considering cultural differences
- Treating psychological principles as rigid rules rather than tendencies with exceptions
- Ignoring actual user research in favor of theoretical principles (test, do not assume)

---

## Frameworks and Methodologies

### 1. Attention and Perception — designing for how people see

1. **Pre-attentive processing** — Color, size, orientation, and motion are processed in under 200ms without conscious effort; use these for critical information
2. **Selective attention** — Users focus on task-relevant information and filter out everything else; do not assume they will see peripheral content
3. **Change blindness** — Users miss changes that occur outside their attention focus; animate transitions to draw attention to changed elements
4. **Peripheral vision** — Low acuity but good at detecting motion; use subtle motion for alerts without disrupting the main task
5. **Gestalt grouping** — Proximity, similarity, enclosure, and continuity determine how elements are perceived as related

### 2. Memory Limitations — designing within cognitive constraints

1. **Working memory capacity** — Recent research shows 4 chunks (Cowan, 2001), not Miller's 7; keep option sets and form steps small
2. **Recognition vs recall** — Seeing triggers memory far more effectively than retrieving from scratch; prefer menus over command-line input
3. **Long-term memory is reconstructive** — Users do not replay exact memories; they reconstruct based on schemas, which means interface training fades quickly
4. **Chunking** — Group related items to reduce cognitive load (phone numbers, credit card numbers, step indicators)
5. **Prospective memory is fragile** — People forget intentions (to save, to click submit); provide reminders and autosave

### 3. Decision-Making Biases — how users actually choose

1. **Anchoring** — The first piece of information disproportionately influences subsequent judgments (pricing tiers, default selections)
2. **Loss aversion** — Losses feel roughly 2x as painful as equivalent gains feel pleasurable; frame actions in terms of what users keep
3. **Choice overload** — Too many options leads to decision paralysis (Iyengar & Lepper, 2000); curate defaults and limit options
4. **Status quo bias** — Users tend to stick with defaults; make the default the best choice for most users
5. **Framing effect** — The same information presented differently leads to different decisions; "95% success rate" vs "5% failure rate"

### 4. Motivation and Engagement — step-by-step application
1. **Identify user goal** — Understand what intrinsic motivation (mastery, autonomy, purpose) or extrinsic motivation (rewards, social status) drives the user
2. **Reduce friction** — Remove unnecessary steps between motivation and action (Fogg Behavior Model: B = MAP)
3. **Provide feedback loops** — Immediate feedback sustains motivation; progress bars, streaks, and completion indicators
4. **Use variable rewards sparingly** — Unpredictable rewards drive engagement but can become manipulative; apply ethically
5. **Support autonomy** — Let users choose their own path; forced flows feel controlling and reduce motivation

---

## How to Apply

### Decision Checklist
- [ ] Does the interface require users to hold more than 4 items in working memory at once?
- [ ] Are interactive elements visually consistent with users' mental models of how they should work?
- [ ] Do important state changes use animation or transitions to avoid change blindness?
- [ ] Are error messages and warnings placed where users are looking, not in peripheral areas?
- [ ] Do form designs use chunking (grouped fields, step indicators) to reduce perceived complexity?
- [ ] Are defaults set to the most common and safest user choice?
- [ ] Is social proof (ratings, testimonials, user counts) presented at decision points?
- [ ] Have dark patterns been eliminated (forced continuity, hidden costs, trick questions)?

### Implementation Patterns

**Miller's Law — Chunking for Readability:**
```html
<!-- Bad: raw 16-digit string overwhelms working memory -->
<input type="text" value="4532015112830366" class="card-input-bad">

<!-- Good: chunked into 4 groups matching mental model of credit cards -->
<div class="card-input-group">
  <input type="text" maxlength="4" pattern="\d{4}" placeholder="1234"
         aria-label="First 4 digits" class="card-chunk">
  <input type="text" maxlength="4" pattern="\d{4}" placeholder="5678"
         aria-label="Middle 4 digits" class="card-chunk">
  <input type="text" maxlength="4" pattern="\d{4}" placeholder="9012"
         aria-label="Next 4 digits" class="card-chunk">
  <input type="text" maxlength="4" pattern="\d{4}" placeholder="3456"
         aria-label="Last 4 digits" class="card-chunk">
</div>

<style>
.card-input-group {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}
.card-chunk {
  width: 4.5rem;
  text-align: center;
  font-size: 1.125rem;
  font-family: 'SF Mono', 'Fira Code', monospace;
  letter-spacing: 0.1em;
  padding: 0.625rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.375rem;
}
.card-chunk:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
}
</style>
```

**Social Proof at Decision Points:**
```html
<!-- Social validation reduces uncertainty at conversion moments -->
<section class="pricing-social-proof">
  <div class="plan-card plan-card--popular">
    <span class="popular-badge">Most Popular</span>
    <h3>Pro Plan</h3>
    <p class="price"><span class="price-amount">$29</span>/month</p>
    <p class="social-indicator">
      <svg class="icon-users" aria-hidden="true">...</svg>
      <strong>12,847 teams</strong> chose this plan this month
    </p>
    <ul class="plan-features">
      <li>Unlimited projects</li>
      <li>Advanced analytics</li>
      <li>Priority support</li>
    </ul>
    <button class="btn-primary">Start Free Trial</button>
  </div>
</section>

<style>
.plan-card--popular {
  border: 2px solid #3b82f6;
  position: relative;
  transform: scale(1.05);
  box-shadow: 0 8px 30px rgba(59, 130, 246, 0.15);
}
.popular-badge {
  position: absolute;
  top: -0.75rem;
  left: 50%;
  transform: translateX(-50%);
  background: #3b82f6;
  color: white;
  padding: 0.25rem 1rem;
  border-radius: 1rem;
  font-size: 0.8125rem;
  font-weight: 600;
}
.social-indicator {
  display: flex;
  align-items: center;
  gap: 0.375rem;
  font-size: 0.875rem;
  color: #64748b;
  margin: 0.75rem 0;
}
</style>
```

**Anchoring in Pricing Display:**
```html
<!-- Anchoring: show the higher price first to make the target price feel reasonable -->
<div class="pricing-anchor">
  <div class="price-comparison">
    <span class="original-price">$99/mo</span>
    <span class="current-price">$49/mo</span>
    <span class="savings-badge">Save 50%</span>
  </div>
  <p class="price-context">Billed annually ($588/year)</p>
</div>

<style>
.price-comparison {
  display: flex;
  align-items: baseline;
  gap: 0.75rem;
}
.original-price {
  font-size: 1.25rem;
  color: #94a3b8;
  text-decoration: line-through;
}
.current-price {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
}
.savings-badge {
  background: #dcfce7;
  color: #16a34a;
  padding: 0.25rem 0.625rem;
  border-radius: 0.25rem;
  font-size: 0.8125rem;
  font-weight: 600;
}
</style>
```

**Progressive Disclosure — Reducing Cognitive Load:**
```html
<!-- Show only what is needed; reveal details on demand -->
<details class="disclosure-section">
  <summary class="disclosure-trigger">
    <span>Advanced Settings</span>
    <svg class="chevron" aria-hidden="true">...</svg>
  </summary>
  <div class="disclosure-content">
    <label>
      <input type="checkbox" checked> Enable email notifications
    </label>
    <label>
      <input type="checkbox"> Send weekly digest
    </label>
    <label>
      Custom domain
      <input type="text" placeholder="app.yourdomain.com">
    </label>
  </div>
</details>

<style>
.disclosure-section {
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  overflow: hidden;
}
.disclosure-trigger {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.25rem;
  cursor: pointer;
  font-weight: 500;
  list-style: none;
  user-select: none;
}
.disclosure-trigger::-webkit-details-marker { display: none; }
.disclosure-content {
  padding: 0 1.25rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}
details[open] .chevron { transform: rotate(180deg); }
.chevron { transition: transform 0.2s ease; }
</style>
```

### Common Mistakes
1. **Citing Miller's 7 as a strict rule** — Modern research (Cowan, 2001) suggests working memory capacity is closer to 4 chunks; and Miller's paper was about discrimination, not UI items
2. **Applying lab findings directly without testing** — Psychology experiments are conducted in controlled conditions that differ from real-world usage; always validate with user testing
3. **Ignoring cultural differences** — Color meanings, reading direction, social proof expectations, and authority perception vary significantly across cultures
4. **Using psychology to justify dark patterns** — Scarcity tactics, forced urgency, and confirmshaming violate user trust and often result in long-term business damage
5. **Overloading with psychology-based features** — Applying every principle simultaneously creates a cluttered, manipulative-feeling interface; select the principles most relevant to each context
6. **Confusing attention-grabbing with good design** — Just because you can draw attention to something does not mean you should; respect the user's primary task

### Integration with Other Topics
- **[Laws of UX (10)](../cognitive-psychology/10-laws-of-ux.md)** — Laws of UX distills psychological principles into named, memorable design laws; this topic provides the deeper scientific basis
- **[Cognitive Load Theory (11)](../cognitive-psychology/11-cognitive-load-theory.md)** — Memory limitations discussed here are the foundation of cognitive load theory's three load types
- **[Gestalt Principles (12)](../cognitive-psychology/12-gestalt-principles.md)** — Gestalt principles are a subset of perceptual psychology covered in the Attention and Perception section above

---

## Resources

### Essential Reading
- Weinschenk, S. (2020). *100 Things Every Designer Needs to Know About People*. New Riders. 2nd edition.
- Johnson, J. (2020). *Designing with the Mind in Mind*. Morgan Kaufmann. 3rd edition.
- Kahneman, D. (2011). *Thinking, Fast and Slow*. Farrar, Straus and Giroux.
- Ariely, D. (2008). *Predictably Irrational*. Harper Perennial.

### Online Resources
- [The Weinschenk Institute — Behavioral Science Resources](https://www.theTeamW.com/)
- [Nielsen Norman Group — Psychology for UX](https://www.nngroup.com/topic/psychology-cognitive-science/)
- [UI Patterns — Psychology-Based Design Patterns](https://ui-patterns.com/)
- [Cognitive Bias Codex — Visual Map of 188 Biases](https://www.visualcapitalist.com/every-single-cognitive-bias/)
- [Laws of UX — Psychological Principles for Designers](https://lawsofux.com/)

### Practice Exercises
1. **Bias audit** — Take a conversion flow from your product (signup, checkout, upgrade) and identify every cognitive bias the design leverages; classify each as ethical (reducing friction) or manipulative (dark pattern)
2. **Memory load test** — Count the number of distinct items a user must hold in working memory at the most complex point in your interface; redesign to keep it under 4
3. **Mental model mapping** — Interview 5 users about how they think your product works; draw their mental models and compare them to the actual system architecture; note mismatches
4. **Change blindness test** — Make a state change in your interface (update a number, swap an element) without animation; ask 5 people to spot the change; measure detection rate before and after adding a transition animation
5. **Default optimization** — List all defaults in your application (form values, settings, notification preferences); for each, determine whether the default matches the most common user preference based on data
