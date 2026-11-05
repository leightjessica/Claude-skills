---
name: policy-presentations
description: >
  Creates policy-oriented PowerPoint (.pptx) presentations designed for non-technical
  audiences such as donors, implementing partners, government officials, and policy
  stakeholders. Use this skill when the user asks for a policy presentation, a stakeholder
  deck, a donor briefing, results dissemination slides, or any presentation that needs to
  communicate research findings to a non-academic audience. Also trigger when the user
  mentions "policy slides", "donor presentation", "dissemination deck", "briefing slides",
  or wants to turn a report or evaluation into a presentation for a non-academic audience.
  Do NOT use this skill for academic seminar talks or conference presentations (use a
  Beamer or academic-slides skill for those).
---

# Policy-Oriented Presentations — Formatting Conventions

This skill produces PowerPoint (.pptx) presentations for policy and donor audiences. These
decks prioritize clarity, visual impact through figures, and concise interpretation. Every
deck must follow the conventions below.

## General Principles

- The audience is non-technical: donors, government officials, implementing partners.
- Show data visually (figures/graphs), not in tables or dense text.
- Each slide should make one clear point that the audience can absorb in 60-90 seconds.
- Aim for roughly one slide per 1-1.5 minutes of speaking time (e.g., 25-35 slides for a
  30-40 minute talk).
- The presenter will narrate and interpret; the slides themselves should be clean and sparse.

## Slide Dimensions and Template

- Use widescreen (16:9) format unless the user provides a template with different dimensions.
- If the user provides a branded template (.pptx), always use it: preserve the background,
  logos, color scheme, and layouts. Apply the template background to all content slides, not
  just the title and closing slides.
- Match the heading color to the template branding (typically the primary brand color from
  the title slide).

## Typography

- **Slide titles/headings:** 24-36pt, bold, in the template brand color. Titles should be
  short and descriptive (not full sentences). No figure numbers in titles (e.g., use
  "Stunting Prevalence" not "Figure 5.4. Stunting Prevalence").
- **Body/bullet text:** 18pt, Arial. Never smaller than 16pt. If text does not fit at 18pt,
  trim the content rather than shrinking the font.
- **Figure caption/source notes:** 8-10pt, gray, positioned at the bottom of figure slides.
  These are the only text elements that may be smaller than 16pt.

## Slide Structure

A policy deck follows this arc:

1. **Title slide** (1 slide): Project name, authors, date. Use the template title layout.
2. **Outline** (1 slide): 6-8 bullet points listing the major sections.
3. **Background and study design** (4-6 slides): Context, intervention description, study
   design, timeline. These are text-based slides with bullets.
4. **Findings sections** (variable): Each major findings domain gets a section header slide
   followed by figure slides. Typically 2-4 sections (e.g., nutrition, livelihoods,
   heterogeneity).
5. **Summary slides** (1 per findings section): A text slide at the end of each findings
   section that synthesizes the key takeaways from that section.
6. **Cost-benefit or policy analysis** (2-4 slides): If relevant, include 1-2 setup slides
   explaining the methodology before showing results, since the audience may not be familiar
   with the approach.
7. **Conclusions and policy implications** (1-2 slides): High-level takeaways and actionable
   recommendations.
8. **Closing slide** (1 slide): "Thank you" with any required disclaimers.

## Figure Slides

Figures are the centerpiece of a policy deck. Follow these rules:

- **One figure per slide.** Never put two graphs on the same slide.
- **Title only, no bullets.** Figure slides use the TITLE_ONLY layout (or equivalent): a
  descriptive title at the top, the graph image centered and filling the available space,
  and a small source/note caption at the bottom.
- **Speaker notes contain the interpretation.** Move all bullet-point interpretation of the
  figure into the slide's speaker notes field. The presenter reads these aloud while the
  audience looks at the graph. Notes should include:
  - The key result (effect size, significance, direction)
  - The comparison group or reference point (e.g., "relative to a control mean of X")
  - Any important caveats or qualifications
- **No figure numbers in titles.** Use descriptive titles like "Stunting Prevalence by
  Treatment Arm" rather than "Figure 5.4. Stunting Prevalence."
- When two consecutive figures share related findings (e.g., HAZ and stunting, consumption
  and poverty), place the combined interpretation notes on the last figure in the sequence
  and add a brief pointer note on earlier figures ("See next slide for combined findings").

## Text/Bullet Slides

- **Maximum 5-6 bullet points per slide.** If there are more points to make, split across
  two slides or consolidate.
- **Each bullet should be one line or at most two short lines at 18pt.** If a bullet wraps
  to three lines, it is too long — rewrite it.
- **No sub-bullets.** If you need hierarchical information, use a two-column layout or
  split across slides. Indented sub-items are hard to read from the back of a room.
- **Use two-column layouts** for comparative information (e.g., treatment arm results
  side-by-side, costs vs. benefits). Font size may be reduced to 16pt in two-column
  layouts but never below 16pt.
- **Every quantitative claim needs a reference point.** Never state "+0.23 SD" without also
  stating what the control group level is or what percentage change this represents.
- **Write bullets as complete sentences and end each with a period.** In general, bullet
  points should read as full sentences and carry terminal punctuation (a period, or a
  question mark for a question). Fragment-style labels that introduce an indented list may
  end with a colon instead (e.g., "We define:"), but standalone bullets should not be left
  without end punctuation.
- **Leave a small, consistent space between bullets.** Match the template's own body style
  rather than packing bullets flush together. If the template master defines a space-before
  (commonly around 10pt), preserve it; do not override it to zero. Do not add large gaps
  either — the goal is a single, even, modest space between every bullet.
- **Left-align titles and anchor them to the top of the title placeholder** (upper-left),
  matching the template. Do not let a short title drift to the vertical center of its box or
  become centered horizontally.

## Section Headers

- Use the SECTION_HEADER layout for major transitions (e.g., "Primary Findings: Nutrition
  Outcomes").
- Keep the section title to 2-4 words on the first line, with an optional subtitle on the
  second line.
- Section headers give the audience a moment to reorient; do not skip them.

## Summary Slides

- Place a summary slide at the end of each major findings section (before the next section
  header).
- Title: "Summary: [Section Name]" (e.g., "Summary: Nutrition Outcomes").
- 5-6 bullets distilling the section's key messages at a higher level than individual figure
  notes.
- These slides serve as natural pause points for questions and ensure the audience retains
  the main takeaways even if they lost track during individual figures.

## Methodology/Setup Slides

- When presenting cost-benefit analysis, econometric approaches, or other technical methods,
  include 1-2 setup slides before the results so the audience understands what they are
  looking at.
- Frame these in plain language: "Do the benefits justify the costs?" rather than "We
  estimate a discounted consumption benefit stream."
- State assumptions clearly and simply (e.g., "We project benefits over 10 years with a 5%
  discount rate").

## Speaker Notes

- Every figure slide MUST have speaker notes with the interpretation.
- Notes should be written as natural talking points, not as an academic paragraph.
- Include the key numbers and significance levels the presenter should mention aloud.
- Structure notes as a short bulleted list (4-6 points) that takes 60-90 seconds to read.

## What NOT to Do

- Do not use font sizes below 16pt for any text the audience needs to read (captions
  excepted).
- Do not put tables of regression coefficients on slides. If tabular results are needed,
  show them as a simple bar chart or coefficient plot.
- Do not use academic jargon without explanation (e.g., say "the program reduces stunting
  by 7 percentage points among the poorest households" rather than "the ITT estimate from
  the four-way interaction specification is -0.07").
- Do not include more than one graph per slide.
- Do not omit section headers between major topics.
- Do not omit summary slides at section transitions.
- Do not put interpretation bullets on the same slide as a figure — use speaker notes.
- Do not include figure numbers (e.g., "Figure 5.4") in slide titles.
- Do not use numbered or bulleted lists with more than 6 items on a single slide.
