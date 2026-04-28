# Sensemaking: Making Sense of Any Mess

## Profile

### What It Is
Sensemaking is the practice of intentionally arranging information so that it becomes understandable and actionable. As defined by Abby Covert, information architecture is "the way we arrange the parts of something to make it understandable as a whole." Her 7-step process provides a practical methodology for tackling any information mess, from organizing a website to structuring an entire business domain.

### Origin and Evolution
- **1999** — Karl Weick publishes *Sensemaking in Organizations*, establishing sensemaking as a field in organizational theory
- **2004-2010** — Information architecture community increasingly focuses on the human side of organizing information, beyond taxonomies and sitemaps
- **2011** — Abby Covert begins presenting "How to Make Sense of Any Mess" at IA conferences, democratizing IA practice
- **2014** — Covert publishes *How to Make Sense of Any Mess*, making IA accessible to non-specialists with a plain-language, step-by-step approach
- **2016-present** — The sensemaking approach gains adoption in content strategy, service design, and organizational design beyond traditional IA

### Key Authors and References
- **Abby Covert** — *How to Make Sense of Any Mess* (2014), founder of the concept of "everyday information architecture"
- **Karl Weick** — *Sensemaking in Organizations* (1995), academic foundation of sensemaking theory
- **Richard Saul Wurman** — *Information Anxiety* (1989), early articulation of the problem of information overload
- **Dan Klyn** — Articulated the ontology-taxonomy-choreography framework for IA
- **Jorge Arango** — *Living in Information* (2018), extending IA to information environments

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Information | Data given context; the arrangement of parts that communicates meaning |
| Architecture | The intentional arrangement of something to serve a purpose |
| Ontology | The meaning we assign to things; what we call them and what they are |
| Taxonomy | The arrangement of things into groups, categories, and hierarchies |
| Choreography | The rules for interaction among the parts; how things flow and connect |
| Mess | Any situation where information is confusing, overwhelming, or poorly organized |
| Controlled Vocabulary | An agreed-upon set of terms used to label and organize content consistently |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Everyone is an information architect** — Arranging information is a fundamental human activity, not just a specialist role; anyone who organizes a spreadsheet, writes a menu, or names a folder is doing IA
2. **The mess is the starting point** — You cannot skip the mess; acknowledging complexity, confusion, and contradiction is the first step toward clarity
3. **Language is the foundation** — How we name things determines how we understand, find, and use them; poor naming creates poor architecture
4. **Intent before structure** — Define what you want to achieve before choosing how to organize; structure without purpose is just rearrangement
5. **Sensemaking is iterative** — The 7 steps are not strictly linear; you will revisit earlier steps as understanding deepens

### When to Use vs. When to Avoid

**Use when:**
- Facing a complex information problem with no clear starting point (new product, content migration, organizational redesign)
- Stakeholders disagree about terminology, categories, or priorities
- Users cannot find content or misunderstand labels and categories
- Merging multiple systems, teams, or content sources into a unified structure
- Teaching non-designers about information architecture for the first time

**Avoid over-application when:**
- The problem is well-defined and the structure is already clear (just implement it)
- You are optimizing a single UI component (use usability heuristics instead)
- The scope is strictly visual design with no structural component
- Analysis paralysis: at some point you must commit to a direction and test it

---

## Frameworks and Methodologies

### 1. The 7 Steps to Making Sense of Any Mess — step-by-step

#### Step 1: Identify the Mess
Acknowledge what is confusing, broken, or overwhelming. Resist the urge to jump to solutions.
- List everything that feels messy or unclear
- Identify who is affected and how
- Document the symptoms: duplicated content, conflicting labels, dead-end navigation, user complaints

#### Step 2: State Your Intent
Define what "good" looks like. What are you trying to achieve? For whom?
- Write a clear intent statement: "We want [users] to be able to [action] so that [outcome]"
- Identify success criteria and constraints
- Align stakeholders on the intent before proceeding

#### Step 3: Face Reality
Audit what exists today. Understand the current state without judgment.
- Conduct a content inventory or information audit
- Map existing structures, labels, and flows
- Identify gaps, redundancies, and contradictions
- Talk to users and stakeholders about their experience

#### Step 4: Choose a Direction
Based on your intent and reality, commit to an organizational approach.
- Decide on ontology: what things are called and what they mean
- Decide on taxonomy: how things are grouped and categorized
- Decide on choreography: how things flow and connect
- Make explicit decisions about what to include and exclude

#### Step 5: Measure the Distance
Determine how far the current state is from the desired state.
- Compare current IA to intended IA
- Identify the biggest gaps and highest-impact changes
- Prioritize what to tackle first based on effort vs. impact
- Define metrics for progress (findability, task completion, user satisfaction)

#### Step 6: Play with Structure
Experiment with different arrangements. Use diagrams, card sorts, and prototypes.
- Create multiple structural options (not just one)
- Test with users through card sorting, tree testing, or prototype navigation
- Iterate on the structure based on feedback
- Document the chosen structure as sitemaps, flow diagrams, or taxonomies

#### Step 7: Prepare to Adjust
Plan for change. No IA is permanent; build in mechanisms for evolution.
- Define governance: who can add, change, or remove content and categories?
- Create style guides and controlled vocabularies to maintain consistency
- Set up regular review cycles (quarterly IA audits)
- Monitor analytics and search logs for signs of structural decay

### 2. Ontology-Taxonomy-Choreography Framework
1. **Ontology** — Define the meaning of things: what is a "product" vs. a "service"? What is an "article" vs. a "guide"? Establish shared definitions before categorizing
2. **Taxonomy** — Group and classify: how do products relate to categories? How do articles nest under topics? Build hierarchies, facets, and tagging systems
3. **Choreography** — Define flow and interaction: in what order does a user encounter these things? How do they move between categories? What are the rules of navigation?

### 3. Practical Research Methods for Sensemaking

#### Card Sorting
- **Open card sort** — Users group content items and name the groups themselves; reveals mental models
- **Closed card sort** — Users sort content into predefined categories; validates proposed structure
- **Hybrid card sort** — Predefined categories plus the option to create new ones

#### Affinity Mapping
1. Write each piece of information on a sticky note (physical or digital)
2. Silently cluster related notes into groups
3. Name each group after clustering
4. Discuss, merge, and refine groups as a team
5. Identify themes and hierarchies from the resulting clusters

---

## How to Apply

### Decision Checklist
- [ ] Have you clearly identified and documented the mess (Step 1)?
- [ ] Have you written an intent statement that all stakeholders agree on (Step 2)?
- [ ] Have you audited the current state without jumping to solutions (Step 3)?
- [ ] Have you explicitly defined ontology (what things mean), taxonomy (how they are grouped), and choreography (how they flow)?
- [ ] Have you tested your structure with real users through card sorting or tree testing (Step 6)?
- [ ] Have you established governance rules for maintaining the IA over time (Step 7)?
- [ ] Have you defined metrics to measure whether the IA is working?

### Implementation Patterns

**Controlled Vocabulary Definition (JSON for CMS/API):**
```json
{
  "controlled_vocabulary": {
    "name": "Content Types",
    "description": "Agreed-upon content type definitions for the knowledge base",
    "terms": [
      {
        "preferred_label": "Tutorial",
        "definition": "Step-by-step instructional content that guides the user through a specific task from start to finish",
        "scope_note": "Must include prerequisites, numbered steps, and expected outcome",
        "use_for": ["How-to", "Walkthrough", "Step-by-step guide"],
        "related": ["Reference", "Example"]
      },
      {
        "preferred_label": "Reference",
        "definition": "Comprehensive documentation of APIs, parameters, configuration options, or specifications",
        "scope_note": "Must be exhaustive within its scope; not instructional",
        "use_for": ["API docs", "Spec sheet", "Technical reference"],
        "related": ["Tutorial", "Concept"]
      },
      {
        "preferred_label": "Concept",
        "definition": "Explanatory content that helps the user understand a topic, pattern, or architectural decision",
        "scope_note": "Focuses on understanding, not on doing; may include diagrams",
        "use_for": ["Explainer", "Overview", "Background"],
        "related": ["Tutorial", "Reference"]
      }
    ]
  }
}
```

**Card Sort Results Visualization:**
```html
<div class="card-sort-results">
  <h2>Open Card Sort Results — 12 Participants</h2>
  <div class="cluster-grid">
    <div class="cluster">
      <h3 class="cluster__name">Getting Started</h3>
      <span class="cluster__agreement">Agreement: 83%</span>
      <ul class="cluster__items">
        <li>Installation guide</li>
        <li>Quick start tutorial</li>
        <li>System requirements</li>
        <li>First project setup</li>
      </ul>
    </div>
    <div class="cluster">
      <h3 class="cluster__name">Configuration</h3>
      <span class="cluster__agreement">Agreement: 67%</span>
      <ul class="cluster__items">
        <li>Environment variables</li>
        <li>Build settings</li>
        <li>Plugin options</li>
      </ul>
    </div>
    <div class="cluster">
      <h3 class="cluster__name">Troubleshooting</h3>
      <span class="cluster__agreement">Agreement: 91%</span>
      <ul class="cluster__items">
        <li>Common errors</li>
        <li>FAQ</li>
        <li>Debug mode</li>
        <li>Error codes reference</li>
      </ul>
    </div>
  </div>
</div>

<style>
.cluster-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
  padding: 1rem 0;
}
.cluster {
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  padding: 1.25rem;
  background: #fafafa;
}
.cluster__name {
  margin: 0 0 0.25rem;
  font-size: 1.1rem;
}
.cluster__agreement {
  display: inline-block;
  font-size: 0.8rem;
  padding: 0.15rem 0.5rem;
  background: #d1fae5;
  color: #065f46;
  border-radius: 9999px;
  margin-bottom: 0.75rem;
}
.cluster__items {
  list-style: none;
  padding: 0;
  margin: 0;
}
.cluster__items li {
  padding: 0.4rem 0.75rem;
  margin: 0.25rem 0;
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 4px;
  font-size: 0.9rem;
}
</style>
```

**Affinity Map Board (HTML/CSS):**
```html
<div class="affinity-board">
  <div class="affinity-column" data-group="ungrouped">
    <h3 class="affinity-column__title">Ungrouped</h3>
    <div class="affinity-card" draggable="true">User cannot find pricing page</div>
    <div class="affinity-card" draggable="true">Search returns irrelevant results</div>
    <div class="affinity-card" draggable="true">Navigation labels are jargon-heavy</div>
  </div>
  <div class="affinity-column" data-group="findability">
    <h3 class="affinity-column__title">Findability Issues</h3>
  </div>
  <div class="affinity-column" data-group="labeling">
    <h3 class="affinity-column__title">Labeling Issues</h3>
  </div>
</div>

<style>
.affinity-board {
  display: flex;
  gap: 1rem;
  padding: 1rem;
  overflow-x: auto;
  min-height: 300px;
}
.affinity-column {
  min-width: 220px;
  background: #f3f4f6;
  border-radius: 8px;
  padding: 1rem;
}
.affinity-column__title {
  font-size: 0.85rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.03em;
  margin-bottom: 0.75rem;
}
.affinity-card {
  background: #fffbeb;
  border: 1px solid #fbbf24;
  border-radius: 4px;
  padding: 0.75rem;
  margin-bottom: 0.5rem;
  cursor: grab;
  font-size: 0.875rem;
}
</style>
```

### Common Mistakes
1. **Jumping to structure without understanding the mess** — The most common mistake is skipping Steps 1-3 and going directly to drawing sitemaps; without understanding the problem space, structure is arbitrary
2. **Treating naming as trivial** — Ontology (what we call things) is the hardest and most important part of IA; teams that do not invest in defining terms end up with inconsistent, confusing labels
3. **Designing for one audience only** — A taxonomy that works for power users may overwhelm beginners; sensemaking requires understanding multiple audiences and creating layered access
4. **Building and abandoning** — Creating an IA once and never revisiting it; without governance (Step 7), the structure decays as new content is added without regard for the existing architecture
5. **Confusing personal organization with shared understanding** — What makes sense to the person who created the structure may be opaque to others; always validate with users through card sorting and tree testing

### Integration with Other Topics
- **[17-information-architecture]** — Morville and Rosenfeld provide the 4 IA systems (organization, labeling, navigation, search) that implement the structures discovered through sensemaking
- **[30-content-strategy]** — Content strategy determines what content exists and why; sensemaking determines how that content is organized and made understandable
- **[13-elements-of-ux]** — Garrett's Structure and Skeleton planes are where sensemaking outcomes get implemented as IA and navigation design

---

## Resources

### Essential Reading
- Covert, A. — *How to Make Sense of Any Mess* (CreateSpace, 2014)
- Weick, K. — *Sensemaking in Organizations* (SAGE Publications, 1995)
- Arango, J. — *Living in Information: Responsible Design for Digital Places* (Two Waves Books, 2018)
- Wurman, R.S. — *Information Anxiety 2* (Que, 2001)
- Klyn, D. — "Explaining Information Architecture" (Understanding Group, available online)

### Online Resources
- [How to Make Sense of Any Mess (howtomakesenseofanymess.com)](http://www.howtomakesenseofanymess.com) — The companion website with free access to core concepts
- [The Understanding Group (understandinggroup.com)](https://understandinggroup.com) — Dan Klyn's IA consultancy with articles on ontology, taxonomy, choreography
- [Optimal Workshop](https://www.optimalworkshop.com) — Tools for card sorting and tree testing exercises
- [World IA Day (worldiaday.org)](https://worldiaday.org) — Annual global event celebrating and advancing IA practice

### Practice Exercises
1. **Personal mess audit** — Choose something in your daily life that is disorganized (email inbox, file system, bookmarks); walk through all 7 steps to reorganize it, documenting each step
2. **Ontology workshop** — Gather 3-5 colleagues and define 10 key terms for your product (e.g., what is a "project" vs. a "workspace" vs. a "folder"?); write one-sentence definitions and identify synonyms to eliminate
3. **Card sort sprint** — Create 30-40 cards representing content items from a real project; run an open card sort with 5 participants using Optimal Workshop, then analyze the results to identify natural groupings
4. **Before-and-after navigation** — Take a website with poor IA; document its current structure, run a tree test to measure findability, redesign the structure using the 7 steps, and tree-test again to measure improvement
