# Data Visualization

## Profile

### What It Is
The practice of encoding quantitative and qualitative information into visual representations that reveal patterns, trends, and relationships in data. Edward Tufte's foundational work establishes that excellent data visualization maximizes the data conveyed per unit of visual element while maintaining graphical integrity. These principles guide web developers in choosing appropriate chart types, eliminating decorative noise, and building accessible, responsive visualizations.

### Origin and Evolution
- **1786** — William Playfair invents the bar chart and line chart in *The Commercial and Political Atlas*
- **1858** — Florence Nightingale creates her polar area diagram to visualize causes of mortality in the Crimean War
- **1967** — Jacques Bertin publishes *Semiology of Graphics*, formalizing visual variable theory (position, size, shape, color, value, orientation, texture)
- **1983** — Edward Tufte publishes *The Visual Display of Quantitative Information*, introducing the data-ink ratio and chartjunk concepts
- **1990** — Tufte publishes *Envisioning Information*, expanding into multivariate data and small multiples
- **2001** — Tufte releases the second edition of *The Visual Display*, refining principles for the digital age
- **2011** — Mike Bostock releases D3.js, making web-native data visualization broadly accessible
- **2020s** — Observable Plot, Chart.js 4, and native CSS/SVG approaches simplify responsive, accessible chart creation

### Key Authors and References
- **Edward Tufte** — *The Visual Display of Quantitative Information* (1983, 2nd ed. 2001)
- **Edward Tufte** — *Envisioning Information* (1990)
- **Edward Tufte** — *Visual Explanations* (1997)
- **Jacques Bertin** — *Semiology of Graphics* (1967, English translation 1983)
- **Stephen Few** — *Show Me the Numbers* (2004, 2nd ed. 2012)
- **Alberto Cairo** — *The Truthful Art* (2016)
- **Mike Bostock** — D3.js library and Observable platform

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Data-Ink Ratio | The proportion of a graphic's ink devoted to non-redundant display of data; maximize it |
| Chartjunk | Decorative elements (3D effects, gradients, unnecessary gridlines) that do not convey data |
| Small Multiples | A series of similar charts using the same scale and axes to enable direct comparison |
| Graphical Integrity | The visual representation must not distort the underlying data (no truncated axes, misleading areas) |
| Lie Factor | The ratio of the effect shown in the graphic to the effect in the data; should be close to 1.0 |
| Visual Encoding | Mapping data dimensions to visual properties: position, length, angle, area, color, saturation |
| Pre-attentive Processing | Visual properties (color, size, orientation) the brain processes in under 250ms without focused attention |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Above all else, show the data** — Every design decision should serve the goal of communicating data clearly; begin by removing everything that does not help the viewer understand the information
2. **Maximize the data-ink ratio** — Within reason, erase non-data ink (excessive gridlines, borders, backgrounds) and redundant data ink (3D effects, ornamental shadings) so the viewer focuses on what matters
3. **Maintain graphical integrity** — The visual representation of numbers must be directly proportional to the numerical quantities represented; never truncate axes or use area encodings that inflate perceived differences
4. **Use small multiples for comparison** — When comparing categories or time periods, use a series of similarly structured charts rather than cramming everything onto one overloaded graphic
5. **Respect the viewer's intelligence** — Provide context, label clearly, and trust the audience to read a well-designed chart without heavy-handed annotation or oversimplification

### When to Use vs. When to Avoid

**Use when:**
- Presenting quantitative data that has patterns, trends, or comparisons to reveal
- Dashboards where users need to monitor multiple metrics at a glance
- Reports where a table of numbers would overwhelm but a chart would clarify
- Exploratory data analysis where visual patterns guide further investigation
- Communicating data to stakeholders who need quick comprehension

**Avoid over-application when:**
- The dataset is very small (fewer than 5 data points) and a well-formatted number or sentence suffices
- Exact values matter more than patterns (use a table instead)
- The audience needs to export or manipulate the raw data
- Adding a chart purely for visual appeal without a clear analytical purpose

---

## Frameworks and Methodologies

### 1. Chart Type Selection — by data relationship
1. **Comparison across categories** — Use bar charts (vertical or horizontal). Horizontal bars work better when category labels are long
2. **Change over time** — Use line charts. Multiple lines for comparison, area fills for cumulative totals
3. **Part-to-whole relationships** — Use stacked bar charts or treemaps. Avoid pie charts when comparing more than 3-4 segments
4. **Distribution** — Use histograms, box plots, or violin plots to show spread and concentration
5. **Correlation between variables** — Use scatter plots. Add a trend line when the relationship is meaningful
6. **Geographic data** — Use choropleth maps or cartograms. Ensure the color scale is perceptually uniform
7. **Hierarchical data** — Use treemaps or sunburst diagrams for nested categories

### 2. Tufte's Data-Ink Audit — for refining existing charts
1. Start with the default chart output from your library or tool
2. Remove the chart border and background fill
3. Reduce gridlines to light, thin lines or remove them entirely (rely on axis labels)
4. Remove 3D effects, gradients, and drop shadows
5. Remove the legend if the series can be directly labeled on the chart
6. Remove redundant axis labels (if the title states the unit, the axis does not need to repeat it)
7. Evaluate the result: if it is now harder to read, selectively restore only the elements that aid comprehension

### 3. Accessibility-First Color Strategy — for inclusive charts
1. Choose a colorblind-safe palette (avoid red/green pairs; use blue/orange or blue/red-purple)
2. Do not encode meaning in color alone; pair color with pattern, shape, or direct labels
3. Test with a color vision deficiency simulator (e.g., Sim Daltonism, Chrome DevTools)
4. Ensure all text in the visualization meets WCAG AA contrast ratios (4.5:1 for normal text)
5. Provide a text alternative (table or description) for screen reader users

---

## How to Apply

### Decision Checklist
- [ ] Is the chart type appropriate for the data relationship being shown (comparison, trend, distribution, correlation, part-to-whole)?
- [ ] Does the chart have a clear, descriptive title?
- [ ] Are axes labeled with units and do they start at zero (or clearly indicate otherwise)?
- [ ] Have you removed unnecessary gridlines, borders, backgrounds, and decorative effects?
- [ ] Is color used accessibly (colorblind-safe palette, not the sole encoding)?
- [ ] Are data series directly labeled instead of requiring a separate legend?
- [ ] Does the chart maintain graphical integrity (lie factor close to 1.0)?
- [ ] Is the visualization responsive and readable on mobile screens?
- [ ] Is there a text alternative or data table for screen reader users?

### Implementation Patterns

**Semantic bar chart with CSS Grid (no JS library required):**
```html
<figure class="bar-chart" role="img" aria-label="Quarterly revenue in thousands: Q1 120, Q2 185, Q3 145, Q4 210">
  <figcaption class="chart-title">Quarterly Revenue (in $K)</figcaption>
  <div class="chart-grid">
    <div class="bar" style="--value: 120; --max: 250;">
      <span class="bar-fill"></span>
      <span class="bar-label">Q1</span>
      <span class="bar-value">$120K</span>
    </div>
    <div class="bar" style="--value: 185; --max: 250;">
      <span class="bar-fill"></span>
      <span class="bar-label">Q2</span>
      <span class="bar-value">$185K</span>
    </div>
    <div class="bar" style="--value: 145; --max: 250;">
      <span class="bar-fill"></span>
      <span class="bar-label">Q3</span>
      <span class="bar-value">$145K</span>
    </div>
    <div class="bar" style="--value: 210; --max: 250;">
      <span class="bar-fill"></span>
      <span class="bar-label">Q4</span>
      <span class="bar-value">$210K</span>
    </div>
  </div>
</figure>
```
```css
.bar-chart {
  --chart-height: 200px;
  max-width: 400px;
  margin: 0 auto;
}

.chart-title {
  font-weight: 600;
  margin-bottom: 1rem;
  text-align: center;
}

.chart-grid {
  display: flex;
  align-items: flex-end;
  gap: 1rem;
  height: var(--chart-height);
  border-bottom: 2px solid #374151;
  padding-bottom: 0.5rem;
}

.bar {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100%;
  justify-content: flex-end;
}

.bar-fill {
  width: 100%;
  height: calc((var(--value) / var(--max)) * 100%);
  background-color: #2563eb;
  border-radius: 4px 4px 0 0;
  transition: height 0.5s ease;
}

.bar-value {
  font-size: 0.75rem;
  font-weight: 600;
  margin-bottom: 0.25rem;
  order: -1;
}

.bar-label {
  font-size: 0.85rem;
  color: #6b7280;
  margin-top: 0.5rem;
}
```

**SVG sparkline for inline trend display:**
```html
<span class="sparkline" aria-label="Trend: rising from 10 to 45 over 7 days">
  <svg viewBox="0 0 100 30" preserveAspectRatio="none" width="80" height="24">
    <polyline
      fill="none"
      stroke="#2563eb"
      stroke-width="2"
      stroke-linecap="round"
      stroke-linejoin="round"
      points="0,28 17,22 33,25 50,18 67,12 83,14 100,2"
    />
  </svg>
</span>
```

**Colorblind-safe palette as CSS custom properties:**
```css
:root {
  /* IBM Design colorblind-safe palette */
  --chart-color-1: #648fff; /* blue */
  --chart-color-2: #fe6100; /* orange */
  --chart-color-3: #dc267f; /* magenta */
  --chart-color-4: #785ef0; /* purple */
  --chart-color-5: #ffb000; /* gold */

  /* Semantic chart colors */
  --chart-positive: #198038;
  --chart-negative: #da1e28;
  --chart-neutral: #6b7280;
}
```

**Responsive chart container:**
```css
.chart-container {
  width: 100%;
  max-width: 800px;
  aspect-ratio: 16 / 9;
  container-type: inline-size;
}

/* Simplify chart on small containers */
@container (max-width: 400px) {
  .chart-container .axis-label {
    font-size: 0.7rem;
  }

  .chart-container .gridline:nth-child(odd) {
    display: none; /* show fewer gridlines */
  }

  .chart-container .legend {
    flex-direction: column;
    gap: 0.25rem;
  }
}
```

### Common Mistakes
1. **Using pie charts for comparison** — Humans are poor at comparing angles and areas; a simple bar chart almost always outperforms a pie chart for comparing values across categories
2. **Truncating the y-axis** — Starting the y-axis at a non-zero value exaggerates differences and distorts perception; if you must truncate, break the axis visually and note it explicitly
3. **Encoding data only in color** — Approximately 8% of men and 0.5% of women have some form of color vision deficiency; always pair color with another encoding (pattern, shape, label)
4. **Overloading a single chart** — Cramming too many data series onto one chart creates clutter; use small multiples or an interactive filter instead
5. **Using 3D effects** — 3D bars, pies, and line charts distort data perception (back bars appear smaller, pie slices at the rear look thinner); always use 2D representations
6. **Missing context** — A chart without a title, axis labels, or units forces the viewer to guess what they are looking at

### Integration with Other Topics
- **25-visual-hierarchy** — Visual hierarchy principles determine how viewers scan a chart: title first, then axes, then data, then annotations
- **26-color-theory** — Color palette selection, contrast, and semantic meaning directly affect chart readability and accessibility
- **32-wcag-accessibility** — WCAG 1.1.1 (text alternatives), 1.4.1 (use of color), and 1.4.3 (contrast) apply directly to data visualizations

---

## Resources

### Essential Reading
- Tufte, E. (2001). *The Visual Display of Quantitative Information*, 2nd Edition. Graphics Press.
- Tufte, E. (1990). *Envisioning Information*. Graphics Press.
- Few, S. (2012). *Show Me the Numbers*, 2nd Edition. Analytics Press.
- Cairo, A. (2016). *The Truthful Art*. New Riders.

### Online Resources
- [Data-to-Viz](https://www.data-to-viz.com/) — Decision tree for choosing the right chart type based on data structure
- [Chartability](https://chartability.fizz.studio/) — Accessibility audit methodology for data visualizations
- [ColorBrewer 2.0](https://colorbrewer2.org/) — Colorblind-safe and print-friendly color schemes for maps and charts
- [Observable Plot](https://observablehq.com/plot/) — High-level JS library for exploratory data visualization
- [WCAG: Use of Color](https://www.w3.org/WAI/WCAG21/Understanding/use-of-color.html) — Guidelines for accessible color use in charts

### Practice Exercises
1. **Data-ink audit**: Take a chart from a dashboard or report you work with. Systematically remove every non-data element (gridlines, background, borders, legend). Selectively add back only what is needed for comprehension. Compare before and after
2. **Chart type challenge**: Given a dataset of monthly sales by product category over two years, create three different chart types (line, grouped bar, small multiples). Evaluate which communicates the story most effectively and articulate why
3. **Accessibility retrofit**: Take an existing chart that uses color alone to distinguish series. Retrofit it with direct labels, patterns, and ARIA attributes so it passes Chartability's heuristics
4. **CSS-only visualization**: Build a horizontal bar chart using only HTML and CSS (no JavaScript). Use CSS custom properties for the values and ensure it is responsive down to 320px width
