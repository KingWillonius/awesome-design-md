# DESIGN.md Platform Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a web platform that hosts, serves, and lets users discover curated DESIGN.md files extracted from real websites so AI agents can generate pixel-perfect UI.

**Architecture:** A Next.js App Router site with a static file store of DESIGN.md documents, a searchable index page, and per-site detail pages that render the markdown and serve raw downloads. No database — content lives in the repo as markdown files with frontmatter metadata.

**Tech Stack:** Next.js 15 (App Router), TypeScript, Tailwind CSS, `gray-matter` for frontmatter parsing, `react-markdown` for rendering, Vercel for deployment.

---

## File Structure

```
src/
  app/
    page.tsx                        # Homepage — searchable grid of all sites
    [slug]/
      page.tsx                      # Individual site detail page
    api/
      [slug]/
        design-md/route.ts          # Raw DESIGN.md download endpoint
  lib/
    designs.ts                      # Load + parse all design entries from /content
    types.ts                        # Shared TypeScript types
  components/
    DesignCard.tsx                  # Card component for homepage grid
    SearchBar.tsx                   # Client-side search/filter input
    MarkdownRenderer.tsx            # Renders DESIGN.md content
content/
  stripe/
    DESIGN.md                       # Frontmatter + full design system markdown
  vercel/
    DESIGN.md
  ... (one folder per site)
```

---

## Task 1: Project Scaffold

**Files:**
- Create: `package.json`, `tsconfig.json`, `tailwind.config.ts`, `next.config.ts`

- [ ] **Step 1: Scaffold Next.js app**

```bash
npx create-next-app@latest . --typescript --tailwind --app --no-src-dir --import-alias "@/*"
```

- [ ] **Step 2: Install dependencies**

```bash
npm install gray-matter react-markdown
npm install -D @types/node
```

- [ ] **Step 3: Verify dev server starts**

```bash
npm run dev
```
Expected: server running at `http://localhost:3000`

- [ ] **Step 4: Commit**

```bash
git add .
git commit -m "chore: scaffold Next.js project"
```

---

## Task 2: TypeScript Types

**Files:**
- Create: `src/lib/types.ts`

- [ ] **Step 1: Write the failing test**

Create `src/lib/__tests__/types.test.ts`:

```typescript
import type { DesignEntry } from "../types";

describe("DesignEntry type", () => {
  it("accepts a valid design entry shape", () => {
    const entry: DesignEntry = {
      slug: "stripe",
      name: "Stripe",
      description: "Payment infrastructure. Signature purple gradients.",
      category: "Fintech & Crypto",
      content: "# Stripe Design System\n...",
    };
    expect(entry.slug).toBe("stripe");
    expect(entry.category).toBe("Fintech & Crypto");
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/lib/__tests__/types.test.ts
```
Expected: FAIL — `Cannot find module '../types'`

- [ ] **Step 3: Create the types file**

```typescript
// src/lib/types.ts
export interface DesignEntry {
  slug: string;
  name: string;
  description: string;
  category: string;
  content: string;
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/lib/__tests__/types.test.ts
```
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/lib/types.ts src/lib/__tests__/types.test.ts
git commit -m "feat: add DesignEntry type"
```

---

## Task 3: Content Files (Sample Data)

**Files:**
- Create: `content/stripe/DESIGN.md`
- Create: `content/vercel/DESIGN.md`
- Create: `content/linear.app/DESIGN.md`

- [ ] **Step 1: Create Stripe DESIGN.md**

```markdown
---
name: Stripe
description: Payment infrastructure. Signature purple gradients, weight-300 elegance.
category: Fintech & Crypto
---

# Stripe Design System

## Visual Theme & Atmosphere
Clean, developer-focused. Dominated by purple (#635BFF) gradients on white surfaces.
Dense documentation layout with strong typographic hierarchy.

## Color Palette
| Name | Hex | Role |
|------|-----|------|
| Stripe Purple | #635BFF | Primary CTA, links, brand accent |
| Dark | #0A2540 | Body text, headings |
| Surface | #FFFFFF | Page background |
| Muted | #F6F9FC | Card backgrounds, section fills |

## Typography
- Display: Sohne, weight 600–700
- Body: Sohne, weight 300–400, 16px/1.6
- Code: Roboto Mono, 14px

## Component Stylings
### Buttons
- Primary: bg #635BFF, text white, radius 6px, px-6 py-3, hover bg #4F49D4
- Secondary: border 1.5px #635BFF, text #635BFF, bg transparent

### Cards
- bg #F6F9FC, border 1px #E3E8EE, radius 8px, p-6, shadow-sm

## Layout Principles
- Max content width: 1200px, centered
- Section padding: 96px vertical
- Grid: 12-column, 24px gutter

## Do's and Don'ts
- DO use weight-300 for large display text — it reads as premium
- DON'T use more than two purple tones in a single section
- DON'T use pure black (#000000) — use #0A2540 instead
```

- [ ] **Step 2: Create Vercel DESIGN.md**

```markdown
---
name: Vercel
description: Frontend deployment platform. Black and white precision, Geist font.
category: Developer Tools & IDEs
---

# Vercel Design System

## Visual Theme & Atmosphere
Radical black-and-white minimalism. Zero color noise — every element earns its place.
High information density balanced with generous whitespace.

## Color Palette
| Name | Hex | Role |
|------|-----|------|
| Black | #000000 | Primary background, headings |
| White | #FFFFFF | Body text on dark, surface on light |
| Gray 100 | #FAFAFA | Card backgrounds |
| Gray 400 | #888888 | Secondary text, borders |
| Gray 900 | #111111 | Subtle dark surfaces |

## Typography
- All text: Geist (variable), fallback: Inter
- Display: weight 700, letter-spacing -0.04em
- Body: weight 400, 15px/1.7
- Code: Geist Mono

## Component Stylings
### Buttons
- Primary: bg #000, text #FFF, radius 6px, px-4 py-2, hover bg #333
- Secondary: border 1px #333, text #000, bg transparent, hover bg #FAFAFA

### Cards
- bg #FAFAFA, border 1px #EAEAEA, radius 8px, p-6, no shadow

## Do's and Don'ts
- DO use negative space — whitespace is a design element
- DON'T add color for decoration — only for semantic meaning (errors, success)
- DON'T use font weights below 400 in body copy
```

- [ ] **Step 3: Create Linear DESIGN.md**

```markdown
---
name: Linear
description: Project management for engineers. Ultra-minimal, precise, purple accent.
category: Productivity & SaaS
---

# Linear Design System

## Visual Theme & Atmosphere
Engineering precision meets aesthetic restraint. Dark surfaces, tight spacing,
every pixel deliberate. Purple accent used sparingly for maximum impact.

## Color Palette
| Name | Hex | Role |
|------|-----|------|
| Surface | #1C1C1E | Primary background |
| Surface Alt | #2C2C2E | Card, panel backgrounds |
| Purple | #5E6AD2 | Primary accent, active states |
| Text | #F5F5F5 | Primary text |
| Text Muted | #8E8E93 | Secondary text, labels |
| Border | #3A3A3C | Dividers, input borders |

## Typography
- UI: Inter, weight 400/500
- Display: Inter, weight 600, letter-spacing -0.02em
- Mono: JetBrains Mono

## Component Stylings
### Buttons
- Primary: bg #5E6AD2, text white, radius 6px, px-3 py-1.5, text-sm
- Ghost: text #8E8E93, no border, hover text #F5F5F5

## Do's and Don'ts
- DO use the purple accent for exactly one interactive element per view
- DON'T use borders heavier than 1px
- DON'T use shadows — elevation is expressed through color contrast only
```

- [ ] **Step 4: Commit**

```bash
git add content/
git commit -m "feat: add sample DESIGN.md content files"
```

---

## Task 4: Content Loader

**Files:**
- Create: `src/lib/designs.ts`
- Create: `src/lib/__tests__/designs.test.ts`

- [ ] **Step 1: Write the failing tests**

```typescript
// src/lib/__tests__/designs.test.ts
import { getAllDesigns, getDesignBySlug } from "../designs";

describe("getAllDesigns", () => {
  it("returns an array of DesignEntry objects", async () => {
    const designs = await getAllDesigns();
    expect(Array.isArray(designs)).toBe(true);
    expect(designs.length).toBeGreaterThan(0);
  });

  it("each entry has required fields", async () => {
    const designs = await getAllDesigns();
    designs.forEach((d) => {
      expect(d.slug).toBeTruthy();
      expect(d.name).toBeTruthy();
      expect(d.category).toBeTruthy();
      expect(d.content).toBeTruthy();
    });
  });
});

describe("getDesignBySlug", () => {
  it("returns a DesignEntry for a valid slug", async () => {
    const entry = await getDesignBySlug("stripe");
    expect(entry).not.toBeNull();
    expect(entry!.slug).toBe("stripe");
    expect(entry!.name).toBe("Stripe");
  });

  it("returns null for an unknown slug", async () => {
    const entry = await getDesignBySlug("nonexistent-site");
    expect(entry).toBeNull();
  });
});
```

- [ ] **Step 2: Run tests to verify they fail**

```bash
npx jest src/lib/__tests__/designs.test.ts
```
Expected: FAIL — `Cannot find module '../designs'`

- [ ] **Step 3: Implement the content loader**

```typescript
// src/lib/designs.ts
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import type { DesignEntry } from "./types";

const CONTENT_DIR = path.join(process.cwd(), "content");

export async function getAllDesigns(): Promise<DesignEntry[]> {
  const slugs = fs.readdirSync(CONTENT_DIR).filter((dir) => {
    const fullPath = path.join(CONTENT_DIR, dir);
    return (
      fs.statSync(fullPath).isDirectory() &&
      fs.existsSync(path.join(fullPath, "DESIGN.md"))
    );
  });

  return slugs.map((slug) => parseDesign(slug)).filter(Boolean) as DesignEntry[];
}

export async function getDesignBySlug(slug: string): Promise<DesignEntry | null> {
  const filePath = path.join(CONTENT_DIR, slug, "DESIGN.md");
  if (!fs.existsSync(filePath)) return null;
  return parseDesign(slug);
}

function parseDesign(slug: string): DesignEntry | null {
  const filePath = path.join(CONTENT_DIR, slug, "DESIGN.md");
  try {
    const raw = fs.readFileSync(filePath, "utf-8");
    const { data, content } = matter(raw);
    return {
      slug,
      name: data.name ?? slug,
      description: data.description ?? "",
      category: data.category ?? "Uncategorized",
      content,
    };
  } catch {
    return null;
  }
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
npx jest src/lib/__tests__/designs.test.ts
```
Expected: PASS (all 4 tests)

- [ ] **Step 5: Commit**

```bash
git add src/lib/designs.ts src/lib/__tests__/designs.test.ts
git commit -m "feat: add content loader with getAllDesigns and getDesignBySlug"
```

---

## Task 5: DesignCard Component

**Files:**
- Create: `src/components/DesignCard.tsx`
- Create: `src/components/__tests__/DesignCard.test.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/components/__tests__/DesignCard.test.tsx
import { render, screen } from "@testing-library/react";
import DesignCard from "../DesignCard";

const mockEntry = {
  slug: "stripe",
  name: "Stripe",
  description: "Payment infrastructure. Signature purple gradients.",
  category: "Fintech & Crypto",
  content: "",
};

describe("DesignCard", () => {
  it("renders the site name", () => {
    render(<DesignCard entry={mockEntry} />);
    expect(screen.getByText("Stripe")).toBeInTheDocument();
  });

  it("renders the description", () => {
    render(<DesignCard entry={mockEntry} />);
    expect(screen.getByText(/Payment infrastructure/)).toBeInTheDocument();
  });

  it("renders a link to the detail page", () => {
    render(<DesignCard entry={mockEntry} />);
    const link = screen.getByRole("link");
    expect(link).toHaveAttribute("href", "/stripe");
  });

  it("renders the category badge", () => {
    render(<DesignCard entry={mockEntry} />);
    expect(screen.getByText("Fintech & Crypto")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/components/__tests__/DesignCard.test.tsx
```
Expected: FAIL — `Cannot find module '../DesignCard'`

- [ ] **Step 3: Implement the component**

```tsx
// src/components/DesignCard.tsx
import Link from "next/link";
import type { DesignEntry } from "@/lib/types";

export default function DesignCard({ entry }: { entry: DesignEntry }) {
  return (
    <Link
      href={`/${entry.slug}`}
      className="block border border-gray-200 rounded-lg p-5 hover:border-gray-400 transition-colors"
    >
      <div className="flex items-start justify-between gap-2 mb-2">
        <h2 className="font-semibold text-gray-900">{entry.name}</h2>
        <span className="text-xs bg-gray-100 text-gray-500 rounded px-2 py-0.5 whitespace-nowrap">
          {entry.category}
        </span>
      </div>
      <p className="text-sm text-gray-600 leading-relaxed">{entry.description}</p>
    </Link>
  );
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/components/__tests__/DesignCard.test.tsx
```
Expected: PASS (all 4 tests)

- [ ] **Step 5: Commit**

```bash
git add src/components/DesignCard.tsx src/components/__tests__/DesignCard.test.tsx
git commit -m "feat: add DesignCard component"
```

---

## Task 6: SearchBar Component

**Files:**
- Create: `src/components/SearchBar.tsx`
- Create: `src/components/__tests__/SearchBar.test.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/components/__tests__/SearchBar.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import SearchBar from "../SearchBar";

describe("SearchBar", () => {
  it("renders a search input", () => {
    render(<SearchBar value="" onChange={() => {}} />);
    expect(screen.getByPlaceholderText(/search/i)).toBeInTheDocument();
  });

  it("calls onChange with new value when user types", () => {
    const onChange = jest.fn();
    render(<SearchBar value="" onChange={onChange} />);
    fireEvent.change(screen.getByPlaceholderText(/search/i), {
      target: { value: "stripe" },
    });
    expect(onChange).toHaveBeenCalledWith("stripe");
  });

  it("displays the current value", () => {
    render(<SearchBar value="linear" onChange={() => {}} />);
    expect(screen.getByDisplayValue("linear")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/components/__tests__/SearchBar.test.tsx
```
Expected: FAIL — `Cannot find module '../SearchBar'`

- [ ] **Step 3: Implement the component**

```tsx
// src/components/SearchBar.tsx
"use client";

export default function SearchBar({
  value,
  onChange,
}: {
  value: string;
  onChange: (val: string) => void;
}) {
  return (
    <input
      type="text"
      placeholder="Search design systems..."
      value={value}
      onChange={(e) => onChange(e.target.value)}
      className="w-full border border-gray-300 rounded-lg px-4 py-2 text-sm focus:outline-none focus:border-gray-500"
    />
  );
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/components/__tests__/SearchBar.test.tsx
```
Expected: PASS (all 3 tests)

- [ ] **Step 5: Commit**

```bash
git add src/components/SearchBar.tsx src/components/__tests__/SearchBar.test.tsx
git commit -m "feat: add SearchBar component"
```

---

## Task 7: Homepage

**Files:**
- Modify: `src/app/page.tsx`
- Create: `src/app/__tests__/page.test.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/app/__tests__/page.test.tsx
import { render, screen } from "@testing-library/react";
import HomePage from "../page";
import * as designs from "@/lib/designs";

jest.mock("@/lib/designs");

const mockDesigns = [
  { slug: "stripe", name: "Stripe", description: "Payment infra.", category: "Fintech & Crypto", content: "" },
  { slug: "vercel", name: "Vercel", description: "Deployment platform.", category: "Developer Tools", content: "" },
];

describe("HomePage", () => {
  beforeEach(() => {
    (designs.getAllDesigns as jest.Mock).mockResolvedValue(mockDesigns);
  });

  it("renders the page heading", async () => {
    render(await HomePage());
    expect(screen.getByText(/DESIGN\.md/i)).toBeInTheDocument();
  });

  it("renders a card for each design", async () => {
    render(await HomePage());
    expect(screen.getByText("Stripe")).toBeInTheDocument();
    expect(screen.getByText("Vercel")).toBeInTheDocument();
  });

  it("shows the total count", async () => {
    render(await HomePage());
    expect(screen.getByText(/2/)).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/app/__tests__/page.test.tsx
```
Expected: FAIL

- [ ] **Step 3: Implement the homepage**

```tsx
// src/app/page.tsx
import { getAllDesigns } from "@/lib/designs";
import DesignCard from "@/components/DesignCard";

export default async function HomePage() {
  const designs = await getAllDesigns();

  return (
    <main className="max-w-5xl mx-auto px-6 py-16">
      <div className="mb-12">
        <h1 className="text-3xl font-bold text-gray-900 mb-3">
          Awesome DESIGN.md
        </h1>
        <p className="text-gray-500 text-sm">
          {designs.length} design systems — drop one in your project, tell your AI agent to use it.
        </p>
      </div>

      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
        {designs.map((entry) => (
          <DesignCard key={entry.slug} entry={entry} />
        ))}
      </div>
    </main>
  );
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/app/__tests__/page.test.tsx
```
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/app/page.tsx src/app/__tests__/page.test.tsx
git commit -m "feat: implement homepage with design grid"
```

---

## Task 8: MarkdownRenderer Component

**Files:**
- Create: `src/components/MarkdownRenderer.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/components/__tests__/MarkdownRenderer.test.tsx
import { render, screen } from "@testing-library/react";
import MarkdownRenderer from "../MarkdownRenderer";

describe("MarkdownRenderer", () => {
  it("renders markdown headings as HTML headings", () => {
    render(<MarkdownRenderer content="## Color Palette" />);
    expect(screen.getByRole("heading", { name: "Color Palette" })).toBeInTheDocument();
  });

  it("renders markdown tables", () => {
    const table = `| Name | Hex |\n|------|-----|\n| Black | #000 |`;
    render(<MarkdownRenderer content={table} />);
    expect(screen.getByText("Black")).toBeInTheDocument();
    expect(screen.getByText("#000")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/components/__tests__/MarkdownRenderer.test.tsx
```
Expected: FAIL

- [ ] **Step 3: Implement the component**

```tsx
// src/components/MarkdownRenderer.tsx
import ReactMarkdown from "react-markdown";

export default function MarkdownRenderer({ content }: { content: string }) {
  return (
    <div className="prose prose-sm prose-gray max-w-none">
      <ReactMarkdown>{content}</ReactMarkdown>
    </div>
  );
}
```

- [ ] **Step 4: Install prose plugin**

```bash
npm install @tailwindcss/typography
```

Add to `tailwind.config.ts`:
```typescript
plugins: [require("@tailwindcss/typography")],
```

- [ ] **Step 5: Run test to verify it passes**

```bash
npx jest src/components/__tests__/MarkdownRenderer.test.tsx
```
Expected: PASS

- [ ] **Step 6: Commit**

```bash
git add src/components/MarkdownRenderer.tsx src/components/__tests__/MarkdownRenderer.test.tsx tailwind.config.ts package.json package-lock.json
git commit -m "feat: add MarkdownRenderer component"
```

---

## Task 9: Site Detail Page

**Files:**
- Create: `src/app/[slug]/page.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/app/[slug]/__tests__/page.test.tsx
import { render, screen } from "@testing-library/react";
import DesignPage, { generateStaticParams } from "../page";
import * as designs from "@/lib/designs";

jest.mock("@/lib/designs");

const mockEntry = {
  slug: "stripe",
  name: "Stripe",
  description: "Payment infrastructure.",
  category: "Fintech & Crypto",
  content: "## Color Palette\n\nStripe purple: #635BFF",
};

describe("DesignPage", () => {
  beforeEach(() => {
    (designs.getDesignBySlug as jest.Mock).mockResolvedValue(mockEntry);
    (designs.getAllDesigns as jest.Mock).mockResolvedValue([mockEntry]);
  });

  it("renders the site name as a heading", async () => {
    render(await DesignPage({ params: { slug: "stripe" } }));
    expect(screen.getByRole("heading", { name: "Stripe" })).toBeInTheDocument();
  });

  it("renders the markdown content", async () => {
    render(await DesignPage({ params: { slug: "stripe" } }));
    expect(screen.getByText(/Color Palette/i)).toBeInTheDocument();
  });

  it("shows a download link", async () => {
    render(await DesignPage({ params: { slug: "stripe" } }));
    expect(screen.getByRole("link", { name: /download/i })).toBeInTheDocument();
  });
});

describe("generateStaticParams", () => {
  it("returns a slug entry per design", async () => {
    const params = await generateStaticParams();
    expect(params).toEqual([{ slug: "stripe" }]);
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/app/\\[slug\\]/__tests__/page.test.tsx
```
Expected: FAIL

- [ ] **Step 3: Implement the detail page**

```tsx
// src/app/[slug]/page.tsx
import { notFound } from "next/navigation";
import { getAllDesigns, getDesignBySlug } from "@/lib/designs";
import MarkdownRenderer from "@/components/MarkdownRenderer";

export async function generateStaticParams() {
  const designs = await getAllDesigns();
  return designs.map((d) => ({ slug: d.slug }));
}

export default async function DesignPage({
  params,
}: {
  params: { slug: string };
}) {
  const entry = await getDesignBySlug(params.slug);
  if (!entry) notFound();

  return (
    <main className="max-w-3xl mx-auto px-6 py-16">
      <div className="mb-8">
        <a href="/" className="text-sm text-gray-400 hover:text-gray-600">
          ← All designs
        </a>
      </div>

      <div className="flex items-start justify-between mb-8">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">{entry.name}</h1>
          <p className="text-gray-500 text-sm mt-1">{entry.description}</p>
        </div>
        <a
          href={`/api/${entry.slug}/design-md`}
          download="DESIGN.md"
          className="text-sm border border-gray-300 rounded px-3 py-1.5 hover:bg-gray-50"
        >
          Download DESIGN.md
        </a>
      </div>

      <MarkdownRenderer content={entry.content} />
    </main>
  );
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/app/\\[slug\\]/__tests__/page.test.tsx
```
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/app/[slug]/page.tsx src/app/[slug]/__tests__/page.test.tsx
git commit -m "feat: add site detail page with markdown rendering"
```

---

## Task 10: Raw Download API Route

**Files:**
- Create: `src/app/api/[slug]/design-md/route.ts`

- [ ] **Step 1: Write the failing test**

```typescript
// src/app/api/[slug]/design-md/__tests__/route.test.ts
import { GET } from "../route";
import * as designs from "@/lib/designs";

jest.mock("@/lib/designs");

describe("GET /api/[slug]/design-md", () => {
  it("returns the raw DESIGN.md content with correct headers", async () => {
    (designs.getDesignBySlug as jest.Mock).mockResolvedValue({
      slug: "stripe",
      name: "Stripe",
      description: "",
      category: "",
      content: "# Stripe Design System",
    });

    const response = await GET({} as Request, { params: { slug: "stripe" } });

    expect(response.status).toBe(200);
    expect(response.headers.get("Content-Type")).toBe("text/markdown");
    expect(response.headers.get("Content-Disposition")).toContain("DESIGN.md");
    const text = await response.text();
    expect(text).toContain("# Stripe Design System");
  });

  it("returns 404 for unknown slug", async () => {
    (designs.getDesignBySlug as jest.Mock).mockResolvedValue(null);
    const response = await GET({} as Request, { params: { slug: "ghost" } });
    expect(response.status).toBe(404);
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/app/api/\\[slug\\]/design-md/__tests__/route.test.ts
```
Expected: FAIL

- [ ] **Step 3: Implement the route**

```typescript
// src/app/api/[slug]/design-md/route.ts
import { NextResponse } from "next/server";
import { getDesignBySlug } from "@/lib/designs";

export async function GET(
  _req: Request,
  { params }: { params: { slug: string } }
) {
  const entry = await getDesignBySlug(params.slug);
  if (!entry) return NextResponse.json({ error: "Not found" }, { status: 404 });

  return new Response(entry.content, {
    status: 200,
    headers: {
      "Content-Type": "text/markdown",
      "Content-Disposition": `attachment; filename="DESIGN.md"`,
    },
  });
}
```

- [ ] **Step 4: Run test to verify it passes**

```bash
npx jest src/app/api/\\[slug\\]/design-md/__tests__/route.test.ts
```
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add src/app/api/[slug]/design-md/route.ts src/app/api/[slug]/design-md/__tests__/route.test.ts
git commit -m "feat: add raw DESIGN.md download API route"
```

---

## Task 11: Client-Side Search on Homepage

**Files:**
- Create: `src/app/DesignGrid.tsx` (client component — search state lives here)
- Modify: `src/app/page.tsx`

- [ ] **Step 1: Write the failing test**

```typescript
// src/app/__tests__/DesignGrid.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import DesignGrid from "../DesignGrid";

const mockDesigns = [
  { slug: "stripe", name: "Stripe", description: "Payment infra.", category: "Fintech & Crypto", content: "" },
  { slug: "vercel", name: "Vercel", description: "Deployment platform.", category: "Developer Tools", content: "" },
  { slug: "linear", name: "Linear", description: "Project management.", category: "Productivity", content: "" },
];

describe("DesignGrid", () => {
  it("renders all cards initially", () => {
    render(<DesignGrid designs={mockDesigns} />);
    expect(screen.getByText("Stripe")).toBeInTheDocument();
    expect(screen.getByText("Vercel")).toBeInTheDocument();
    expect(screen.getByText("Linear")).toBeInTheDocument();
  });

  it("filters cards when user types in search", () => {
    render(<DesignGrid designs={mockDesigns} />);
    fireEvent.change(screen.getByPlaceholderText(/search/i), {
      target: { value: "stripe" },
    });
    expect(screen.getByText("Stripe")).toBeInTheDocument();
    expect(screen.queryByText("Vercel")).not.toBeInTheDocument();
    expect(screen.queryByText("Linear")).not.toBeInTheDocument();
  });

  it("shows empty state when nothing matches", () => {
    render(<DesignGrid designs={mockDesigns} />);
    fireEvent.change(screen.getByPlaceholderText(/search/i), {
      target: { value: "zzznomatch" },
    });
    expect(screen.getByText(/no results/i)).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npx jest src/app/__tests__/DesignGrid.test.tsx
```
Expected: FAIL

- [ ] **Step 3: Implement DesignGrid**

```tsx
// src/app/DesignGrid.tsx
"use client";

import { useState } from "react";
import DesignCard from "@/components/DesignCard";
import SearchBar from "@/components/SearchBar";
import type { DesignEntry } from "@/lib/types";

export default function DesignGrid({ designs }: { designs: DesignEntry[] }) {
  const [query, setQuery] = useState("");

  const filtered = designs.filter((d) => {
    const q = query.toLowerCase();
    return (
      d.name.toLowerCase().includes(q) ||
      d.description.toLowerCase().includes(q) ||
      d.category.toLowerCase().includes(q)
    );
  });

  return (
    <div>
      <div className="mb-6">
        <SearchBar value={query} onChange={setQuery} />
      </div>

      {filtered.length === 0 ? (
        <p className="text-gray-400 text-sm">No results for "{query}".</p>
      ) : (
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          {filtered.map((entry) => (
            <DesignCard key={entry.slug} entry={entry} />
          ))}
        </div>
      )}
    </div>
  );
}
```

- [ ] **Step 4: Update homepage to use DesignGrid**

```tsx
// src/app/page.tsx
import { getAllDesigns } from "@/lib/designs";
import DesignGrid from "./DesignGrid";

export default async function HomePage() {
  const designs = await getAllDesigns();

  return (
    <main className="max-w-5xl mx-auto px-6 py-16">
      <div className="mb-10">
        <h1 className="text-3xl font-bold text-gray-900 mb-2">
          Awesome DESIGN.md
        </h1>
        <p className="text-gray-500 text-sm">
          {designs.length} design systems — copy one into your project.
        </p>
      </div>
      <DesignGrid designs={designs} />
    </main>
  );
}
```

- [ ] **Step 5: Run tests to verify they pass**

```bash
npx jest src/app/__tests__/DesignGrid.test.tsx
```
Expected: PASS (all 3 tests)

- [ ] **Step 6: Commit**

```bash
git add src/app/DesignGrid.tsx src/app/page.tsx src/app/__tests__/DesignGrid.test.tsx
git commit -m "feat: add client-side search filtering to homepage"
```

---

## Task 12: Final Verification

- [ ] **Step 1: Run full test suite**

```bash
npx jest --coverage
```
Expected: All tests pass, coverage ≥ 80%

- [ ] **Step 2: Build production bundle**

```bash
npm run build
```
Expected: Build succeeds with no errors

- [ ] **Step 3: Smoke test locally**

```bash
npm run start
```
Open `http://localhost:3000` — verify:
- Grid of design cards loads
- Search filters cards as you type
- Clicking a card opens the detail page
- Markdown renders correctly
- Download link returns a `.md` file

- [ ] **Step 4: Final commit**

```bash
git add .
git commit -m "chore: final build verification"
```
