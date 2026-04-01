---
name: presentation
description: "Create polished HTML presentation decks from source documents and publish them to GitHub Pages. Use this skill whenever the user wants to create a presentation, slide deck, pitch deck, case study presentation, or any kind of slides. Also use when the user says 'make me a deck', 'build slides', 'create a presentation', or has source documents they want turned into a visual presentation. Handles the full pipeline: reading source docs, generating a single-file HTML deck, pushing to GitHub, and enabling GitHub Pages."
---

# HTML Presentation Deck Generator

You create production-grade, single-file HTML presentation decks from source documents in the working directory, then publish them live via GitHub Pages.

## When to Use

- User asks to create a presentation, deck, or slides
- User has source documents (PDF, markdown, text) they want turned into slides
- User wants to present information visually for a meeting, interview, or pitch
- User says anything like "make a deck", "build slides", "create a presentation"

## Workflow Overview

1. **Read source documents** in the working directory
2. **Plan the slide structure** with the user
3. **Generate the HTML presentation** using the frontend-design skill
4. **Verify** all slides fit on one screen, render correctly
5. **Publish** to GitHub and enable GitHub Pages

---

## Step 1: Read Source Documents

Scan the working directory for all content files:
- PDFs (use the Read tool with pages parameter for large PDFs)
- Markdown files (.md)
- Text files (.txt)
- Any other readable documents

Read every document thoroughly. Extract:
- Key themes, data points, metrics, and quotes
- The narrative arc — what story do these documents tell?
- Specific numbers, percentages, and results that make good visual callouts
- Any structural hints (e.g., "5 slides", section headings, bullet points)

## Step 2: Plan the Slide Structure

Before generating anything, present the user with a proposed slide structure:
- Slide count (default 5-6 unless user specifies)
- Title and key content for each slide
- Which data points / metrics go where
- Any diagrams or visual elements planned

Wait for user confirmation or adjustments before proceeding.

## Step 3: Generate the HTML Presentation

Invoke the **frontend-design** skill with a detailed prompt containing all slide content. The prompt must specify every piece of content for every slide — do not leave anything for the frontend-design skill to invent.

### Design Standards (Non-negotiable)

Every presentation must follow these rules. They exist because presentations are shown on projectors and shared screens where scrolling is impossible and inconsistency is distracting.

**Layout:**
- Single HTML file with all CSS and JS embedded (no external dependencies except Google Fonts)
- Every slide must fit within one viewport — no scrolling on any slide. Set `overflow: hidden` on slides.
- Content centered horizontally with `max-width: 1200px`
- Top-aligned content with consistent `40px` top padding across all slides
- All slides use the same color mode (light mode: off-white/white backgrounds, navy text)

**Navigation:**
- Arrow key navigation (left/right, up/down)
- Spacebar advances to next slide
- Home/End keys jump to first/last slide
- Clickable dot indicators at the bottom (transparent background, no pill/container — just floating dots)
- Do NOT include left/right arrow buttons on the sides of the page
- Touch swipe support for mobile
- Slide counter in top-right corner (e.g., "3 / 6")

**Nav Dots Styling:**
The dot indicators must have a fully transparent background — no pill, no backdrop-filter, no container border. Just bare dots floating at the bottom center. The dot colors should naturally match the overall palette:
- Inactive dots: use `--text-light` at ~30% opacity
- Active dot: use `--accent` with a subtle glow (`box-shadow: 0 0 10px` using accent color at ~40% opacity)
```css
.nav-dots {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 12px;
  z-index: 100;
  background: transparent;  /* no background */
  border: none;
  padding: 0;
}
```

**Animation:**
- All components on a slide appear simultaneously — no staggered entry animations
- Use CSS transitions (not keyframe animations) for the fade between slides: `opacity` and `visibility` with `transition: opacity 0.7s`
- Simple `.anim` class with `.anim.visible` toggled by JS on slide change

**Typography:**
```html
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;1,9..40,300;1,9..40,400&display=swap" rel="stylesheet">
```
- Headings: DM Serif Display (serif)
- Body text: DM Sans (sans-serif)
- Section labels: 11px uppercase, letter-spacing 2px, accent color

**Color Palette:**

Do NOT use a fixed color palette. Instead, derive the color scheme from the source documents:
- If the documents reference a company or brand, research that brand's colors and use them as the primary/accent palette
- If the documents are about a product, use colors that evoke the product's industry or domain
- If no clear brand context exists, choose a sophisticated palette that matches the tone of the content (e.g., warm earth tones for organic topics, cool blues for tech, bold contrasts for startups)

Define all colors as CSS variables in `:root` for consistency. Every palette needs at minimum:
- `--primary`: main heading/text color (dark)
- `--accent`: highlight color for labels, borders, key callouts
- `--accent-light`: lighter variant of accent for glows and backgrounds
- `--accent-glow`: very transparent accent for subtle backgrounds (rgba, ~0.15 alpha)
- `--bg`: slide background (light)
- `--bg-alt`: alternate background for contrast sections
- `--card-bg`: card background color
- `--card-border`: card border color
- `--text`: body text color (medium gray)
- `--text-light`: secondary text color (lighter gray)

All slides must use the same color mode. Maintain sufficient contrast for readability.
Cards should have `border-radius: 14px` and hover effects: `translateY(-3px)` with subtle box-shadow.

**Slide Engine (CSS):**
```css
.slide {
  position: absolute !important;  /* !important needed to prevent override */
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.7s cubic-bezier(0.22, 1, 0.36, 1), visibility 0.7s;
  overflow: hidden;
}
.slide.active { opacity: 1; visibility: visible; }
.slide-content {
  width: 100%;
  max-width: 1200px;
  padding: 40px 60px 0;
}
```

**Slide Engine (JS pattern):**
```javascript
function goTo(idx) {
  if (idx < 0 || idx >= total || idx === current) return;
  slides[current].querySelectorAll('.anim').forEach(el => el.classList.remove('visible'));
  slides[current].classList.remove('active');
  dots[current].classList.remove('active');
  current = idx;
  slides[current].classList.add('active');
  dots[current].classList.add('active');
  counter.textContent = (current + 1) + ' / ' + total;
  // Trigger entry
  setTimeout(() => {
    slides[current].querySelectorAll('.anim').forEach(el => el.classList.add('visible'));
  }, 50);
}
// Init first slide
setTimeout(() => {
  slides[0].querySelectorAll('.anim').forEach(el => el.classList.add('visible'));
}, 100);
```

### Content Density Guidelines

Presentations are viewed on screens, not read like documents. Each slide must fit in one viewport (~729px height). Follow these density rules:

- **Slide labels:** 11px, 4px margin-bottom
- **Slide titles:** clamp(24px, 2.8vw, 36px), 16px margin-bottom
- **Card padding:** 18-20px
- **Card gaps:** 12-16px
- **Body text:** 11-13px, line-height 1.4-1.5
- **Metric values:** clamp(22px, 2.4vw, 32px) in DM Serif Display
- **Metric labels:** 12.5px uppercase

If a slide has too much content, reduce font sizes and padding rather than allowing scroll. If it still doesn't fit, split into two slides.

### Design Principles

These principles come from iterative user feedback and reflect what makes a presentation feel polished versus rough. Follow them closely.

**Bullet points over arrow chains.** When showing lists, comparisons (before/after), processes, or metrics, use clean bullet points — not inline arrow-connected flows (`A → B → C`). Arrow chains look cluttered and are hard to scan. Bullet points are faster to read and easier to align.

**Grid alignment across stacked sections.** If two sections sit above and below each other (e.g., a Before/After comparison above a Primary/Guardrail metrics section), they must share the exact same CSS grid structure — same `grid-template-columns`, same gap. When columns visually align top-to-bottom, the slide looks intentionally designed. When they don't, it looks sloppy. This is one of the most common sources of "something feels off" feedback.

**Impact-focused subtitles.** The subtitle on a title slide should describe the *outcome and impact* of the work, not the presenter's job title or role. Write it as a statement of what was achieved: "Driving 30% conversion and $62M revenue growth by rebuilding the digital purchasing journey" — not "Product Manager, PLG — Case Study Presentation."

**Plain language in labels.** Avoid vendor names, technical jargon, or internal terminology in visible labels. Write for the audience, not for the engineering spec. Use "Instant credit decision via API" not "Instant credit decision via DealerTrack API." Use "Deposit Conversion Rate" not "Deposit-to-Delivery."

**Remove decorative clutter.** Every visual element should earn its place. If an arrow divider between two cards doesn't add clarity — remove it. If a badge/tag at the top of a title slide just restates the obvious — remove it. If a pill-shaped container around nav dots adds visual noise — make it transparent. When in doubt, remove.

**Generous spacing between content blocks.** Distinct content sections (e.g., stat cards vs. before/after comparison vs. metrics definitions) need visible breathing room between them. Use 16-24px margins between major sections. Cramped slides feel overwhelming even when the content itself is fine.

**Consistent color mode.** All slides must use the same background treatment. Never mix dark-mode slides with light-mode slides in the same deck — it feels jarring and unprofessional. Pick one and commit.

## Step 4: Verify the Presentation

After generating the HTML file, verify it works:

1. If Playwright MCP is available, start a local HTTP server and navigate through all slides:
   - Check that no slide overflows (scrollHeight === clientHeight)
   - Check that `position: absolute` is correctly applied to `.slide` elements
   - Take screenshots for the user to review
2. If Playwright is not available, remind the user to open the file in their browser and check each slide

Fix any issues found during verification before proceeding to publish.

## Step 5: Publish to GitHub Pages

Run these steps in sequence:

```bash
# 1. Initialize git repo (if not already one)
git init

# 2. Create .gitignore
cat > .gitignore << 'GITIGNORE'
*.png
.playwright-mcp/
GITIGNORE

# 3. Stage and commit
git add .gitignore index.html [other-source-files]
git commit -m "Add presentation: [title]

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"

# 4. Create GitHub repo and push
gh repo create [repo-name] --public --source=. --push

# 5. Enable GitHub Pages
gh api repos/[owner]/[repo-name]/pages -X POST --input - <<'EOF'
{"build_type":"legacy","source":{"branch":"master","path":"/"}}
EOF
```

Derive the repo name from the presentation title (lowercase, hyphenated, e.g., "weave-case-study-presentation"). Get the owner from `gh api user -q .login`.

After publishing, tell the user:
- The GitHub repo URL
- The GitHub Pages URL (https://[owner].github.io/[repo-name]/)
- That it may take 1-2 minutes for the first deployment

---

## Common Slide Patterns

Use these as building blocks when designing slides:

**Title Slide:** Large serif title with one accent-colored keyword. Impact-focused subtitle (outcome, not role). 3 hero stat cards with large numbers and uppercase labels. Before/After comparison using side-by-side cards with bullet points (no arrow chains). Primary and guardrail metrics as bulleted lists in a 2-column grid aligned to the same columns as the comparison above.

**Discovery / Analysis Slide:** Two-column layout — funnel/process visualization on the left, card list on the right. Hypothesis bar at bottom with items in a horizontal grid.

**Journey / Flow Slide:** Horizontal phase bar with color-coded sections, product cards below each phase with metric tags. Include a legend for the metric tag colors.

**Strategy / Timeline Slide:** 3 phase cards side-by-side with colored top borders, result callouts in accent-glow boxes. Stakeholder narrative callout in primary color. GTM collaboration summary and team composition at the bottom.

**Results / Metrics Slide:** 3x2 grid of metric tiles (large serif numbers, small uppercase labels). 2x2 grid of learning cards with accent left border. Include one "what surprised me" or self-reflection learning to show intellectual humility.

**Bridge / Application Slide:** Context paragraph referencing specific company knowledge (recent acquisitions, product launches, public metrics). 3 focus-area cards with numbered badges and parallels to the case study. Action-plan box (e.g., "First 90 Days") with concrete steps. Italic closing statement.

---

## Argument Handling

The user may invoke this skill with arguments like:
- `/presentation` — scan docs in current directory, propose structure
- `/presentation "My Topic" 6 slides` — use the topic and slide count
- `/presentation` with prior conversation context — extract requirements from chat history

If no arguments, scan the working directory for source documents and ask the user about the desired structure.

Save the final HTML file as `index.html` in the working directory.
