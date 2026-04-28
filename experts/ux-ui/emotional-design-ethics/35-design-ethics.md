# Design Ethics

## Profile

### What It Is
Design ethics is the practice of making intentional, morally informed choices about how digital products influence user behavior, protect user rights, and serve the broader public good. It encompasses identifying and eliminating dark patterns, designing inclusively for diverse abilities and contexts, embedding privacy by default, and recognizing that designers hold real power over the people who use their products. Ethical design is not an afterthought or compliance checklist but a fundamental lens through which every design decision should be evaluated.

### Origin and Evolution
- **1971** — Victor Papanek publishes *Design for the Real World*, arguing designers have a social and moral responsibility
- **2010** — Harry Brignull coins the term "dark patterns" and launches darkpatterns.org, cataloging deceptive UI practices
- **2012** — Mike Monteiro publishes *Design Is a Job*, arguing that ethics are inseparable from professional design practice
- **2016** — Cennydd Bowles begins writing about the systemic ethical implications of design decisions at scale
- **2018** — GDPR takes effect in the EU, codifying privacy by design into law and forcing designers to rethink consent flows
- **2019** — Mike Monteiro publishes *Ruined by Design*, a forceful call for designer accountability and professional ethics
- **2020** — Cennydd Bowles publishes *Future Ethics*, expanding the ethical frame beyond dark patterns to AI, automation, and societal impact
- **2022-present** — FTC begins enforcement against dark patterns; California and EU strengthen digital rights legislation

### Key Authors and References
- **Mike Monteiro** — *Ruined by Design* (2019)
- **Mike Monteiro** — *Design Is a Job* (2012)
- **Cennydd Bowles** — *Future Ethics* (2020)
- **Harry Brignull** — Dark Patterns research and taxonomy (2010-present)
- **Kat Holmes** — *Mismatch: How Inclusion Shapes Design* (2018)
- **Victor Papanek** — *Design for the Real World* (1971)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Dark Pattern | A deceptive UI design trick that manipulates users into doing things they did not intend |
| Confirmshaming | Using guilt-laden language to dissuade users from declining an offer |
| Roach Motel | A design that makes it easy to get into a situation but extremely hard to get out |
| Privacy Zuckering | Tricking users into sharing more personal data than they intend to |
| Inclusive Design | Designing for the full range of human diversity (ability, language, culture, context) |
| Privacy by Design | Building privacy protections into the architecture from the start, not as an afterthought |
| Informed Consent | Ensuring users clearly understand what they are agreeing to before they agree |
| Design Justice | Centering the perspectives of those most affected by design decisions |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Designers are gatekeepers, not order-takers** — Every design choice either protects or harms users; claiming you were "just following requirements" does not absolve responsibility for the impact of the work
2. **Respect user autonomy** — Users must be able to make informed, uncoerced decisions about their data, their attention, and their money; any design that undermines free choice is a dark pattern
3. **Design for the margins, benefit the center** — When you design for people with disabilities, limited connectivity, or cultural differences, you create solutions that work better for everyone
4. **Privacy is a right, not a feature** — Personal data collection must be minimal, transparent, and consensual; the default must always be the most private option
5. **Measure what matters ethically** — If your success metrics reward manipulation (e.g., signups obtained through confusion, cancellation abandonment rates), your incentives are misaligned with user welfare

### When to Use vs. When to Avoid

**Use when:**
- Designing any flow that involves user consent, data collection, or financial commitment
- Evaluating existing interfaces for dark patterns during UX audits
- Building sign-up, subscription, cancellation, or notification permission flows
- Making decisions about default settings that affect user privacy or behavior
- Working on products used by vulnerable populations (children, elderly, people in crisis)

**Avoid over-application when:**
- Ethical caution becomes paralysis that prevents shipping any product at all
- You impose your own cultural values as universal ethical standards without research
- Theoretical ethical perfection prevents practical improvements (perfect is the enemy of good)
- You weaponize ethics language to block legitimate business decisions that do not actually harm users

---

## Frameworks and Methodologies

### 1. Dark Patterns Taxonomy — identification guide
1. **Confirmshaming** — Guilt-tripping language on decline options ("No thanks, I don't want to save money")
2. **Roach Motel** — Easy to sign up, nearly impossible to cancel (hidden cancellation, phone-only cancellation)
3. **Privacy Zuckering** — Confusing privacy settings that default to maximum data sharing
4. **Bait and Switch** — User sets out to do one thing but a different, undesirable thing happens
5. **Hidden Costs** — Charges that appear only at the final step of checkout after the user is committed
6. **Trick Questions** — Confusing wording with double negatives so users accidentally opt into things
7. **Forced Continuity** — Free trial automatically converts to paid subscription without clear notice
8. **Disguised Ads** — Advertisements designed to look like content or navigation elements
9. **Friend Spam** — Requesting contact access, then messaging contacts without clear user understanding
10. **Misdirection** — Using visual hierarchy to draw attention away from important information toward the preferred action

### 2. Ethical Design Audit — step-by-step
1. **Map all decision points** — Identify every place the user must make a choice (consent, purchase, sharing, settings)
2. **Test the reversal** — For each decision point, check that the opposite action is equally easy and discoverable
3. **Read the microcopy aloud** — If any option sounds manipulative, guilt-laden, or confusing when spoken, rewrite it
4. **Check the defaults** — Ensure default settings serve the user's interest, not the business's; opt-in must be the default for non-essential features
5. **Time the escape** — Measure how long it takes to cancel, unsubscribe, or delete an account; it should take no longer than signing up
6. **Test with vulnerable users** — Include elderly users, non-native speakers, and people with cognitive disabilities in usability testing
7. **Audit business metrics** — If success metrics reward user confusion or inability to cancel, flag them for redesign

### 3. Inclusive Design Framework — systematic approach
1. **Recognize exclusion** — Identify who is being left out by current design decisions (visual, motor, cognitive, situational disabilities)
2. **Learn from diversity** — Involve people with diverse abilities and backgrounds in the research and design process
3. **Solve for one, extend to many** — Designing captions for deaf users also serves users in noisy environments; curb cuts for wheelchairs also help parents with strollers
4. **Test across contexts** — Validate designs on low-end devices, slow connections, small screens, and with assistive technologies

---

## How to Apply

### Decision Checklist
- [ ] Can users complete the same action to leave as they did to join (sign up vs. cancel symmetry)?
- [ ] Are all decline/reject options presented with neutral, respectful language?
- [ ] Do default settings favor user privacy and minimal data collection?
- [ ] Is the total cost visible before the user begins the checkout process?
- [ ] Can users understand consent prompts without legal expertise?
- [ ] Does the design work for users with visual, motor, auditory, and cognitive disabilities?
- [ ] Are notification permission requests contextual and explained, not immediate and unexplained?
- [ ] Is there a clear, accessible way to delete an account and all associated data?
- [ ] Would you be comfortable if a journalist wrote an article describing this design decision?

### Implementation Patterns

**Ethical vs. dark pattern — unsubscribe flow:**
```html
<!-- DARK PATTERN: Confirmshaming and friction -->
<div class="unsubscribe-dark">
  <h2>Are you sure you want to miss out?</h2>
  <p>You'll stop receiving our exclusive deals and VIP content.</p>
  <div class="actions">
    <button class="btn-primary btn-large">Keep my VIP status</button>
    <a href="#" class="shame-link">
      No thanks, I don't like saving money
    </a>
  </div>
</div>

<!-- ETHICAL: Clear, respectful, and reversible -->
<div class="unsubscribe-ethical">
  <h2>Unsubscribe from emails</h2>
  <p>You will stop receiving marketing emails. You can resubscribe at any time from your account settings.</p>
  <fieldset>
    <legend>Choose your preference:</legend>
    <label class="radio-option">
      <input type="radio" name="email-pref" value="reduce">
      Reduce to once a month
    </label>
    <label class="radio-option">
      <input type="radio" name="email-pref" value="unsubscribe" checked>
      Unsubscribe from all marketing emails
    </label>
  </fieldset>
  <div class="actions-balanced">
    <button class="btn-primary" type="submit">Save preference</button>
    <a href="/account" class="btn-text">Go back</a>
  </div>
</div>

<style>
/* Dark pattern styling that manipulates hierarchy */
.shame-link {
  font-size: 0.75rem;
  color: #94a3b8;
  text-decoration: none;
  margin-top: 0.5rem;
  display: block;
}

/* Ethical styling with balanced visual weight */
.actions-balanced {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-top: 1.5rem;
}

.radio-option {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  margin-bottom: 0.5rem;
  cursor: pointer;
  transition: border-color 0.15s ease;
}

.radio-option:hover {
  border-color: #6366f1;
}

.btn-text {
  background: none;
  border: none;
  color: #475569;
  font-size: 1rem;
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 2px;
}
</style>
```

**Ethical consent form — informed and uncoerced:**
```html
<!-- DARK PATTERN: Pre-checked boxes, confusing language -->
<form class="consent-dark">
  <label>
    <input type="checkbox" checked>
    I do not want to not receive promotional emails
  </label>
  <label>
    <input type="checkbox" checked>
    Share my data with trusted partners to improve my experience
  </label>
  <button type="submit">Accept All and Continue</button>
</form>

<!-- ETHICAL: Unchecked by default, plain language, granular control -->
<form class="consent-ethical" aria-labelledby="consent-heading">
  <h3 id="consent-heading">Communication preferences</h3>
  <p class="consent-description">
    Choose what you would like to receive. You can change these at any time in Settings.
  </p>

  <label class="consent-option">
    <input type="checkbox" name="consent-marketing">
    <span class="consent-label">
      <strong>Product updates</strong>
      <span class="consent-detail">New features and improvements, sent monthly</span>
    </span>
  </label>

  <label class="consent-option">
    <input type="checkbox" name="consent-partners">
    <span class="consent-label">
      <strong>Partner offers</strong>
      <span class="consent-detail">Offers from selected third-party partners</span>
    </span>
  </label>

  <div class="consent-actions">
    <button type="submit" class="btn-primary">Save preferences</button>
    <button type="submit" class="btn-secondary" formaction="/skip-consent">
      Skip for now
    </button>
  </div>
</form>

<style>
.consent-ethical {
  max-width: 480px;
  padding: 2rem;
}

.consent-description {
  color: #64748b;
  font-size: 0.9375rem;
  margin-bottom: 1.25rem;
  line-height: 1.5;
}

.consent-option {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  padding: 1rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  margin-bottom: 0.75rem;
  cursor: pointer;
  transition: border-color 0.15s ease, background-color 0.15s ease;
}

.consent-option:hover {
  border-color: #a5b4fc;
  background: #f8fafc;
}

.consent-option input[type="checkbox"] {
  margin-top: 0.25rem;
  width: 1.125rem;
  height: 1.125rem;
  accent-color: #6366f1;
}

.consent-label {
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.consent-detail {
  font-size: 0.8125rem;
  color: #94a3b8;
}

.consent-actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

.btn-secondary {
  background: #fff;
  border: 1px solid #e2e8f0;
  color: #475569;
  padding: 0.625rem 1.25rem;
  border-radius: 0.5rem;
  cursor: pointer;
  font-size: 1rem;
}
</style>
```

**Ethical notification permission — contextual and honest:**
```html
<!-- DARK PATTERN: Immediate, unexplained permission request on page load -->
<script>
  // Fires immediately before the user understands why
  Notification.requestPermission();
</script>

<!-- ETHICAL: Contextual pre-prompt with explanation -->
<div class="notification-prompt" role="dialog" aria-labelledby="notif-title" hidden>
  <div class="notification-prompt__icon" aria-hidden="true">
    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
      <path d="M12 22c1.1 0 2-.9 2-2h-4c0 1.1.9 2 2 2zm6-6v-5c0-3.07-1.63-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.64 5.36 6 7.92 6 11v5l-2 2v1h16v-1l-2-2z" fill="#6366f1"/>
    </svg>
  </div>
  <div class="notification-prompt__content">
    <h3 id="notif-title">Get notified when your order ships</h3>
    <p>We'll send one notification when your package is on its way. No promotional messages.</p>
  </div>
  <div class="notification-prompt__actions">
    <button class="btn-primary btn-sm" onclick="requestNotificationPermission()">
      Notify me
    </button>
    <button class="btn-text btn-sm" onclick="dismissPrompt()">
      Not now
    </button>
  </div>
</div>

<style>
.notification-prompt {
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  max-width: 360px;
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 0.75rem;
  padding: 1.25rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: flex-start;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
}

.notification-prompt__content p {
  color: #64748b;
  font-size: 0.875rem;
  line-height: 1.5;
  margin: 0.25rem 0 0;
}

.notification-prompt__actions {
  width: 100%;
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.btn-sm {
  padding: 0.375rem 1rem;
  font-size: 0.875rem;
}
</style>

<script>
// Only show notification prompt after a meaningful action
function showNotificationPromptAfterPurchase() {
  const prompt = document.querySelector('.notification-prompt');
  if (Notification.permission === 'default') {
    prompt.hidden = false;
  }
}

function requestNotificationPermission() {
  Notification.requestPermission().then(permission => {
    document.querySelector('.notification-prompt').hidden = true;
  });
}

function dismissPrompt() {
  document.querySelector('.notification-prompt').hidden = true;
  // Remember the choice — do not ask again this session
  sessionStorage.setItem('notif-dismissed', 'true');
}
</script>
```

**Account deletion — symmetry with sign-up:**
```html
<!-- Account deletion should be as easy as sign-up -->
<section class="danger-zone">
  <h3>Delete account</h3>
  <p>Permanently remove your account and all associated data. This cannot be undone.</p>
  <button class="btn-danger" onclick="document.getElementById('deleteDialog').showModal()">
    Delete my account
  </button>
</section>

<dialog id="deleteDialog" class="delete-dialog">
  <h3>Confirm account deletion</h3>
  <p>All your data will be permanently removed within 30 days.</p>
  <ul class="deletion-summary">
    <li>12 projects will be deleted</li>
    <li>48 files will be removed</li>
    <li>Your profile will no longer be visible</li>
  </ul>
  <p class="deletion-note">
    You can download a copy of your data before deleting.
    <a href="/account/export">Download my data</a>
  </p>
  <form method="dialog" class="delete-actions">
    <button type="submit" class="btn-secondary" value="cancel">Cancel</button>
    <button type="submit" class="btn-danger" value="confirm">
      Permanently delete account
    </button>
  </form>
</dialog>

<style>
.danger-zone {
  border: 1px solid #fecaca;
  border-radius: 0.75rem;
  padding: 1.5rem;
  margin-top: 2rem;
}

.delete-dialog {
  border: none;
  border-radius: 0.75rem;
  padding: 2rem;
  max-width: 28rem;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.delete-dialog::backdrop {
  background: rgba(0, 0, 0, 0.5);
}

.deletion-summary {
  background: #fef2f2;
  border-radius: 0.5rem;
  padding: 0.75rem 1.25rem;
  margin: 1rem 0;
  color: #991b1b;
  font-size: 0.875rem;
}

.deletion-note {
  font-size: 0.875rem;
  color: #64748b;
}

.delete-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

.btn-danger {
  background: #ef4444;
  color: white;
  border: none;
  padding: 0.625rem 1.25rem;
  border-radius: 0.5rem;
  cursor: pointer;
  font-weight: 600;
}
</style>
```

### Common Mistakes
1. **Treating ethics as a final-stage review** — Ethics must be embedded from the discovery phase onward; retrofitting ethical improvements onto a dark-pattern-dependent business model rarely works
2. **Confusing persuasion with manipulation** — Ethical design can still be persuasive; the difference is whether the user benefits from the action they are being persuaded to take
3. **Hiding behind A/B test results** — A dark pattern that "wins" in conversion metrics does not make it ethical; short-term metric gains from deception erode long-term trust and brand value
4. **Performative inclusion** — Adding a single accessibility pass at the end of a project instead of designing inclusively from the start; true inclusion requires structural participation, not cosmetic fixes
5. **Privacy theater** — Displaying complex cookie consent banners that technically comply with regulation while being deliberately designed to confuse users into accepting all tracking

### Integration with Other Topics
- **[WCAG Accessibility (32)](../accessibility/32-wcag-accessibility.md)** — Accessibility is an ethical imperative, not a feature; inclusive design principles directly overlap with WCAG requirements for perceivable, operable, understandable, and robust interfaces
- **[Emotional Design (34)](./34-emotional-design.md)** — Emotional design techniques can be used ethically to delight or unethically to manipulate; design ethics provides the boundary between the two
- **[Usability Heuristics (05)](../usability-research/05-usability-heuristics.md)** — Nielsen's heuristics for error prevention, user control, and match with the real world are fundamentally ethical principles; violating them to increase conversion is a dark pattern

---

## Resources

### Essential Reading
- Monteiro, M. (2019). *Ruined by Design: How Designers Destroyed the World, and What We Can Do to Fix It*. Mule Books.
- Monteiro, M. (2012). *Design Is a Job*. A Book Apart.
- Bowles, C. (2020). *Future Ethics*. NowNext Press.
- Holmes, K. (2018). *Mismatch: How Inclusion Shapes Design*. MIT Press.
- Brignull, H. (2023). *Deceptive Patterns: Exposing the Tricks Tech Companies Use to Control You*. Testimonium.

### Online Resources
- [Deceptive Design (formerly darkpatterns.org)](https://www.deceptive.design/) — Catalog and taxonomy of dark patterns with real-world examples
- [Ethical Design Handbook by Smashing Magazine](https://ethicaldesignhandbook.com/) — Practical guide for implementing ethical design in teams
- [Microsoft Inclusive Design Toolkit](https://inclusive.microsoft.design/) — Framework and activities for designing inclusively
- [EFF: Privacy by Design](https://www.eff.org/deeplinks/2020/03/privacy-design) — Electronic Frontier Foundation guidance on privacy-respecting systems
- [FTC: Bringing Dark Patterns to Light](https://www.ftc.gov/reports/bringing-dark-patterns-light) — Regulatory perspective on deceptive design

### Practice Exercises
1. **Dark pattern audit**: Select a popular consumer app (a subscription service, social media platform, or e-commerce site). Screenshot and categorize every dark pattern you can find using Brignull's taxonomy. Propose ethical redesigns for the three most harmful patterns
2. **Cancellation symmetry test**: Time how long it takes to sign up for a service versus cancel it. Document every extra step in the cancellation flow that did not exist in signup. Redesign the cancellation flow to achieve parity
3. **Consent redesign workshop**: Take a real cookie consent banner and redesign it for genuine informed consent. Requirements: no pre-checked boxes, equal visual weight for accept and reject, plain language descriptions of each tracking category
4. **Inclusive design sprint**: Choose one feature from your current project. Identify three groups of people who might be excluded by the current design (e.g., screen reader users, users with slow connections, non-native speakers). Redesign the feature to include them, and document how each inclusive improvement benefits all users
