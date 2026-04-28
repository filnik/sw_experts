# Usability Heuristics

## Profile

### What It Is
Jakob Nielsen's 10 Usability Heuristics are broad rules of thumb for interaction design, not specific usability guidelines. Originally developed in 1994, they remain the most widely used framework for evaluating the usability of user interfaces. Heuristic evaluation is a discount usability inspection method where a small set of evaluators examines an interface against these recognized principles.

### Origin and Evolution
- **1990** — Nielsen and Molich publish the original set of heuristics based on factor analysis of 249 usability problems
- **1994** — Nielsen refines the list to the canonical 10 heuristics in "Usability Engineering"
- **1995** — "Severity Ratings for Usability Problems" formalized the 0-4 scale
- **2005-2015** — Heuristics adapted for mobile, responsive, and touch interfaces
- **2020** — Nielsen Norman Group updates heuristic descriptions with modern examples and clarifications

### Key Authors and References
- **Jakob Nielsen** — *Usability Engineering* (1993)
- **Jakob Nielsen & Rolf Molich** — *Heuristic Evaluation of User Interfaces* (1990)
- **Jakob Nielsen** — *10 Usability Heuristics for User Interface Design* (1994, updated 2020)
- **Rolf Molich & Jakob Nielsen** — *Improving a Human-Computer Dialogue* (1990)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Heuristic Evaluation | Inspection method where evaluators judge compliance with heuristics |
| Severity Rating | Scale from 0 (not a problem) to 4 (usability catastrophe) |
| Discount Usability | Cost-effective methods that find most problems with fewer resources |
| Evaluator Effect | Different evaluators find different problems; 3-5 recommended |
| False Alarms | Issues flagged by evaluators that are not real user problems |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Visibility** — Users should always know what is going on through timely and appropriate feedback
2. **Match** — The system should speak the user's language and follow real-world conventions
3. **Freedom** — Users make mistakes; provide clearly marked exits and undo capabilities
4. **Consistency** — Follow platform conventions so users do not have to wonder whether different words or actions mean the same thing
5. **Prevention** — Eliminate error-prone conditions or present confirmation before committing to actions

### When to Use vs. When to Avoid

**Use when:**
- Evaluating an existing interface for usability issues before user testing
- You have limited budget or time and cannot run full usability studies
- Performing expert reviews during design iteration cycles
- Auditing competitor products for strengths and weaknesses
- Training junior designers to develop usability awareness

**Avoid over-application when:**
- You treat heuristic evaluation as a replacement for user testing (it is complementary)
- Evaluators lack domain expertise and produce excessive false positives
- The product requires deep contextual understanding that only real users can provide
- You are designing for highly specialized or expert-level audiences with unique workflows

---

## Frameworks and Methodologies

### 1. The 10 Usability Heuristics — Complete Reference

1. **Visibility of System Status** — Keep users informed about what is going on through appropriate feedback within reasonable time
2. **Match Between System and Real World** — Use words, phrases, and concepts familiar to the user rather than system-oriented terms
3. **User Control and Freedom** — Support undo and redo; provide emergency exits from unwanted states
4. **Consistency and Standards** — Follow platform and industry conventions; do not make users wonder whether different actions mean the same thing
5. **Error Prevention** — Design to prevent problems from occurring; offer confirmation dialogs for critical actions
6. **Recognition Rather Than Recall** — Minimize memory load by making elements, actions, and options visible
7. **Flexibility and Efficiency of Use** — Provide accelerators for expert users without confusing novices
8. **Aesthetic and Minimalist Design** — Remove irrelevant or rarely needed information; every extra unit competes with relevant information
9. **Help Users Recognize, Diagnose, and Recover from Errors** — Error messages should be in plain language, indicate the problem precisely, and suggest solutions
10. **Help and Documentation** — Provide searchable, task-focused documentation even though the system should be usable without it

### 2. Heuristic Evaluation Method — step-by-step
1. **Select 3-5 evaluators** with usability expertise and domain knowledge
2. **Brief evaluators** on the system, user personas, and key tasks
3. **Independent evaluation** — each evaluator inspects the interface alone, noting violations
4. **Record findings** — document each issue with the violated heuristic, location, and description
5. **Assign severity ratings** (0: not a problem, 1: cosmetic, 2: minor, 3: major, 4: catastrophe)
6. **Aggregate findings** — merge all evaluator reports, removing duplicates
7. **Prioritize** — address severity 4 and 3 issues first, then 2 and 1

---

## How to Apply

### Decision Checklist
- [ ] Has the interface been reviewed by at least 3 independent evaluators?
- [ ] Does every interactive element provide visible feedback on state change?
- [ ] Are labels and terminology based on user research, not internal jargon?
- [ ] Can users undo or escape from every significant action?
- [ ] Are visual patterns and interaction behaviors consistent throughout the application?
- [ ] Are error-prone inputs constrained or validated before submission?
- [ ] Can users recognize options from the interface rather than recalling them from memory?
- [ ] Are there keyboard shortcuts or accelerators for frequent tasks?
- [ ] Is every piece of displayed information necessary for the current task?
- [ ] Do error messages explain the problem and suggest a recovery path?

### Implementation Patterns

**Heuristic 1 — Visibility of System Status:**
```html
<!-- Loading state with progress indication -->
<button id="submitBtn" class="btn-primary" aria-busy="false">
  <span class="btn-text">Submit Order</span>
  <span class="btn-loading" hidden>
    <svg class="spinner" aria-hidden="true">...</svg>
    Processing...
  </span>
</button>

<style>
.btn-primary[aria-busy="true"] .btn-text { display: none; }
.btn-primary[aria-busy="true"] .btn-loading { display: inline-flex; align-items: center; gap: 0.5rem; }
.spinner { animation: spin 1s linear infinite; width: 1rem; height: 1rem; }

/* Step progress indicator */
.progress-steps {
  display: flex;
  justify-content: space-between;
  counter-reset: step;
}
.progress-steps li {
  flex: 1;
  text-align: center;
  position: relative;
}
.progress-steps li.completed::before {
  content: "\2713";
  background-color: #22c55e;
  color: white;
}
.progress-steps li.active::before {
  border-color: #3b82f6;
  color: #3b82f6;
  font-weight: bold;
}
</style>
```

**Heuristic 3 — User Control and Freedom:**
```html
<!-- Undo action with timed dismissal -->
<div class="undo-toast" role="alert" aria-live="polite">
  <span>Message deleted.</span>
  <button class="undo-btn" onclick="undoDelete()">Undo</button>
  <div class="undo-timer" style="animation: shrink 8s linear forwards;"></div>
</div>

<style>
.undo-toast {
  position: fixed;
  bottom: 1.5rem;
  left: 50%;
  transform: translateX(-50%);
  background: #1e293b;
  color: white;
  padding: 0.75rem 1.25rem;
  border-radius: 0.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.25);
}
.undo-btn {
  background: none;
  border: 1px solid white;
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 0.25rem;
  cursor: pointer;
  font-weight: 600;
}
.undo-timer {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 3px;
  width: 100%;
  background: #3b82f6;
}
@keyframes shrink { from { width: 100%; } to { width: 0%; } }
</style>
```

**Heuristic 5 — Error Prevention:**
```html
<!-- Confirmation dialog for destructive action -->
<dialog id="confirmDialog" class="confirm-dialog">
  <h3>Delete this project?</h3>
  <p>This will permanently remove <strong>3 files</strong> and <strong>12 tasks</strong>. This action cannot be undone.</p>
  <div class="dialog-actions">
    <button class="btn-secondary" onclick="confirmDialog.close()">Cancel</button>
    <button class="btn-danger" onclick="confirmDelete()">Delete Project</button>
  </div>
</dialog>

<style>
.confirm-dialog {
  border: none;
  border-radius: 0.75rem;
  padding: 2rem;
  max-width: 28rem;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}
.confirm-dialog::backdrop { background: rgba(0,0,0,0.5); }
.dialog-actions { display: flex; justify-content: flex-end; gap: 0.75rem; margin-top: 1.5rem; }
.btn-danger {
  background: #ef4444;
  color: white;
  border: none;
  padding: 0.5rem 1.25rem;
  border-radius: 0.375rem;
  cursor: pointer;
}
</style>
```

**Heuristic 6 — Recognition Rather Than Recall:**
```html
<!-- Recent searches and suggestions reduce memory burden -->
<div class="search-container">
  <input type="search" id="search" placeholder="Search products..."
         autocomplete="off" aria-expanded="false" aria-controls="suggestions">
  <ul id="suggestions" class="suggestions-list" role="listbox" hidden>
    <li class="suggestion-group-label">Recent Searches</li>
    <li role="option" class="suggestion-item">wireless headphones</li>
    <li role="option" class="suggestion-item">usb-c hub</li>
    <li class="suggestion-group-label">Popular</li>
    <li role="option" class="suggestion-item">mechanical keyboard</li>
  </ul>
</div>

<style>
.suggestions-list {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 0 0 0.5rem 0.5rem;
  list-style: none;
  padding: 0;
  margin: 0;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
.suggestion-group-label {
  font-size: 0.75rem;
  color: #64748b;
  padding: 0.5rem 1rem 0.25rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.suggestion-item {
  padding: 0.5rem 1rem;
  cursor: pointer;
}
.suggestion-item:hover { background: #f1f5f9; }
</style>
```

**Heuristic 9 — Help Users Recover from Errors:**
```html
<!-- Clear, constructive error message -->
<div class="form-field error">
  <label for="email">Email Address</label>
  <input type="email" id="email" value="john@gmailcom"
         aria-invalid="true" aria-describedby="email-error">
  <p id="email-error" class="error-message" role="alert">
    This email appears to be missing a dot. Did you mean <button class="suggestion-link"
    onclick="document.getElementById('email').value='john@gmail.com'">john@gmail.com</button>?
  </p>
</div>

<style>
.form-field.error input {
  border-color: #ef4444;
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.1);
}
.error-message {
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.375rem;
  display: flex;
  align-items: center;
  gap: 0.25rem;
}
.suggestion-link {
  background: none;
  border: none;
  color: #2563eb;
  text-decoration: underline;
  cursor: pointer;
  font-size: inherit;
  padding: 0;
}
</style>
```

### Common Mistakes
1. **Using heuristics as a checkbox** — The heuristics are principles for judgment, not binary pass/fail tests; context matters when assessing severity
2. **Single-evaluator reviews** — One person finds only 35% of issues on average; you need 3-5 evaluators to achieve adequate coverage
3. **Ignoring severity ratings** — Treating all violations equally leads to wasted effort on cosmetic issues while critical problems remain
4. **Confusing aesthetic preference with heuristic violation** — A design can be visually different from your preference without violating any heuristic
5. **Skipping the aggregation step** — Evaluators must work independently first; group discussion before evaluation creates bias

### Integration with Other Topics
- **[Design of Everyday Things (01)](../usability-research/01-design-of-everyday-things.md)** — Norman's principles of affordance, feedback, and mapping directly correspond to heuristics 1, 2, and 4
- **[First Principles of Interaction (03)](../usability-research/03-first-principles-interaction.md)** — Provides the theoretical foundation that heuristics operationalize into evaluable criteria
- **[Laws of UX (10)](../cognitive-psychology/10-laws-of-ux.md)** — Psychological laws that explain why each heuristic works from a cognitive science perspective

---

## Resources

### Essential Reading
- Nielsen, J. (1993). *Usability Engineering*. Morgan Kaufmann.
- Nielsen, J. (1994). *Heuristic Evaluation*. In Nielsen, J. & Mack, R.L. (Eds.), Usability Inspection Methods. Wiley.
- Nielsen, J. (2020). *10 Usability Heuristics for User Interface Design* (updated article). NN/g.

### Online Resources
- [Nielsen Norman Group — 10 Usability Heuristics](https://www.nngroup.com/articles/ten-usability-heuristics/)
- [NN/g — How to Conduct a Heuristic Evaluation](https://www.nngroup.com/articles/how-to-conduct-a-heuristic-evaluation/)
- [NN/g — Severity Ratings for Usability Problems](https://www.nngroup.com/articles/how-to-rate-the-severity-of-usability-problems/)
- [NN/g — Usability Heuristics Applied to Video Games](https://www.nngroup.com/articles/usability-heuristics-applied-video-games/)

### Practice Exercises
1. **Heuristic audit of your own project** — Pick one page of an application you are building and evaluate it against all 10 heuristics; assign severity ratings and prioritize the top 3 fixes
2. **Competitive heuristic comparison** — Choose two competing products (e.g., two e-commerce checkouts) and evaluate both against the same heuristic; identify where each excels
3. **Error message rewrite workshop** — Collect 10 error messages from real applications and rewrite each to comply with Heuristic 9 (plain language, precise problem, constructive suggestion)
4. **Cross-evaluator calibration** — Have three colleagues independently evaluate the same interface, then aggregate findings and discuss disagreements in severity ratings
