---
name: Development Standards
description: "Code standards for chicforgeeks_frontend: Modern tech stack, SOLID/DRY principles, small focused components, best practices"
---

# Development Standards — chicforgeeks_frontend

## 🎯 Core Principles

Always apply these principles to **all** code changes:

### 1. **Newest Technology** 🚀

- **React 19+** with App Router (Next.js 15+)
- **TypeScript** with strict mode — leverage `keyof typeof` for type safety
- **React Context** over prop drilling (no Redux for simple state)
- **React.memo** for performance-critical components
- **CSS Modules** for local scoping (no globals)
- **Three.js + React Three Fiber** for 3D rendering
- Use hooks for all state management (no class components)

### 2. **SOLID Principles** 🏗️

| Principle                 | Application                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| **S**ingle Responsibility | One component = one job. If > 200 lines, split it.                                        |
| **O**pen/Closed           | Extend via config, not by modifying core. Example: `thumbnailConfig.ts` for 3D positions. |
| **L**iskov Substitution   | Components with same interface are interchangeable. Props should be predictable.          |
| **I**nterface Segregation | Props should be minimal. Don't pass data a component doesn't use.                         |
| **D**ependency Inversion  | Components depend on abstractions (Context, config objects) not concrete implementations. |

### 3. **DRY (Don't Repeat Yourself)** 🔄

- **No duplicate functions** — consolidate into reusable utilities
- **Centralized config** — use objects like `thumbnailConfig.ts` for repeated values
- **Extract utilities** → `lib/utils/` for shared logic
- **Reuse components** — one `ClothingThumbnail`, not 4 variants (Shirt/Pants/etc.)
- **Shared formatting** → `formatGarmentLabel()` used everywhere labels appear

### 4. **Best Practices** ✅

#### Component Structure

```typescript
// ✅ Good: Small, focused, memoized
export const MyComponent = memo(function MyComponent({ prop }: Props) {
  return <div>{prop}</div>;
});

// ❌ Avoid: Long component with multiple responsibilities
export function MyComponent({ prop1, prop2, prop3, ... }) {
  // 300+ lines with logic, state, effects, rendering
}
```

#### Imports

- `formatGarmentLabel()` for all garment name display
- `THUMBNAIL_POSITIONS` for 3D positioning
- `useGarmentType()` from `GarmentContext` instead of prop drilling

#### TypeScript

```typescript
// ✅ Type-safe garment validation
export const THUMBNAIL_POSITIONS = { shirt: {...}, pants: {...} } as const;
export type GarmentType = keyof typeof THUMBNAIL_POSITIONS;

// ❌ Avoid magic strings
const position = garmentType === "shirt" ? [...] : [...];
```

#### CSS

- Use CSS Modules (`.module.css`)
- Leverage CSS variables (`var(--spacing-md)`, `var(--color-primary)`)
- Reduce padding in compact UIs (`padding: 2px` instead of `var(--spacing-xs)`)

---

## 📏 Component Size Guidelines

| Size       | Lines   | Decision                       |
| ---------- | ------- | ------------------------------ |
| **Small**  | < 100   | Perfect. Keep as-is.           |
| **Medium** | 100–200 | Acceptable if tightly focused. |
| **Large**  | 200–300 | Split into smaller pieces.     |
| **Huge**   | > 300   | Break immediately.             |

### How to Split Components

**Before:**

```typescript
export function VirtualTryOnPage({ ... }) {
  // 350 lines: Canvas setup, clothing grid, sidebar, upload modal
}
```

**After:**

```typescript
export function VirtualTryOnPage({ ... }) {
  return (
    <div>
      <Sidebar />
      <ClothingSection activeTab={activeTab} ... />
      <Canvas3DViewer avatar={avatar} ... />
    </div>
  );
}

// Each child < 150 lines
function Sidebar({ ... }) { ... }
function ClothingSection({ ... }) { ... }
function Canvas3DViewer({ ... }) { ... }
```

---

## 🏛️ Architecture Patterns

### Configuration Centralization

```typescript
// ✅ GOOD: thumbnailConfig.ts used globally
export const THUMBNAIL_POSITIONS = {
  shirt: { position: [0, 0, 1], camera: [0, 1, 2.5], scale: [1, 1, 1] },
  pants: { position: [0, -0.2, 1], camera: [0, 0.5, 2.5], scale: [1, 1, 1] },
} as const;

// Used in:
// - ClothingThumbnail.tsx
// - ClothingGrid.tsx
// - ItemVariationSelector.tsx
```

### Context for Prop Drilling Elimination

```typescript
// ✅ GOOD: GarmentProvider wraps components
<GarmentProvider garmentType="shirt">
  <ClothingGrid /> {/* Gets garmentType from context */}
</GarmentProvider>

// ❌ AVOID: Passing same prop through 5 levels
<ClothingSection garmentType="shirt">
  <ClothingGrid garmentType="shirt">
    <ClothingTabGrid garmentType="shirt">
      <ClothingThumbnail garmentType="shirt" />
    </ClothingTabGrid>
  </ClothingGrid>
</ClothingSection>
```

### Utility Functions

```typescript
// ✅ GOOD: Shared across app
export function formatGarmentLabel(name: string): string {
  // Logic here, used in:
  // - ClothingGrid
  // - ItemVariationSelector
  // - UploadModal
}

// ❌ AVOID: Duplicating formatting logic
const labels = items.map(item => item.name.replace(...).split(...));
```

---

## 🔍 Code Review Checklist

Before committing, verify:

- [ ] Component < 200 lines (split if larger)
- [ ] No duplicate logic (check `lib/utils/`)
- [ ] Types are strict (no `any`, proper unions)
- [ ] Props are minimal (only what's used)
- [ ] Context used instead of prop drilling (if > 2 levels)
- [ ] Memoization for performance (Canvas components, lists)
- [ ] CSS uses variables, no hardcoded values (check `variables.css`)
- [ ] Labels use `formatGarmentLabel()` (not raw names)
- [ ] Config values from `thumbnailConfig.ts` (not hardcoded)
- [ ] Comments explain "why", not "what" (code should be obvious)

---

## 📁 File Organization

```
app/
├── lib/
│   ├── config/          ← Centralized config (thumbnailConfig.ts)
│   ├── context/         ← React Context (GarmentContext.tsx)
│   ├── utils/           ← Shared functions (formatGarmentLabel, etc.)
│   ├── services/        ← API calls and external logic
│   └── models/          ← TypeScript types
├── components/
│   └── 3d/             ← Reusable 3D components (ClothingThumbnail)
└── virtual-try-on/
    └── components/     ← Feature-specific components (max 200 lines each)
```

---

## 🚨 Anti-Patterns to Avoid

❌ **Magic numbers** → Put in config

```typescript
// ❌ BAD
position={[0, 0, 1]}

// ✅ GOOD
position={THUMBNAIL_POSITIONS[garmentType].position}
```

❌ **Prop drilling > 2 levels** → Use Context

```typescript
// ❌ BAD
<A prop={x}><B prop={x}><C prop={x}><D prop={x} /></D></C></B></A>

// ✅ GOOD
<Provider value={x}><A><B><C><D /></C></B></A></Provider>
```

❌ **Hardcoded labels** → Use formatters

```typescript
// ❌ BAD
<p>{item.name.replace(/\.glb/, "")}</p>

// ✅ GOOD
<p>{formatGarmentLabel(item.name)}</p>
```

❌ **Long components** → Split early

```typescript
// ❌ BAD: 400 lines, mixed concerns
export function MyPage() {
  // Canvas setup, state, effects, rendering, styling...
}

// ✅ GOOD: Split responsibilities
export function MyPage() {
  return <><Canvas3D /><Sidebar /><Control /></>;
}
```

❌ **Hardcoded CSS values** → Use CSS variables

```css
/* ❌ BAD: Magic numbers scattered everywhere */
.button {
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 14px;
  gap: 8px;
  margin-bottom: 12px;
}

/* ✅ GOOD: Use centralized variables */
.button {
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-lg);
  font-size: var(--font-size-base);
  gap: var(--spacing-sm);
  margin-bottom: var(--spacing-md);
}
```

**CSS Variable Usage:**

- `--spacing-*`: `xs` (2px), `sm` (4px), `md` (8px), `lg` (12px), `xl` (16px), `2xl` (24px)
- `--radius-*`: `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, `full`
- `--font-size-*`: `xs`, `sm`, `base`, `lg`, `xl`, `2xl`
- `--color-*`: primary, secondary, success, error, border, text, background, etc.
- `--shadow-*`: `sm`, `md`, `lg`, `button-depth`, `button-active`
- `--transition-*`: `normal` (200ms), `slower` (400ms)

Never use hardcoded values like `10px`, `#fff`, `0.95rem` — always check `variables.css` first.

````

---

## 💡 Example: Applying All Principles

**Task:** Display 3D clothing thumbnails in a grid

**✅ SOLID + DRY Solution:**

```typescript
// 1. Config (Open/Closed: extensible without code changes)
export const THUMBNAIL_POSITIONS = { shirt: {...}, pants: {...} } as const;
export type GarmentType = keyof typeof THUMBNAIL_POSITIONS;

// 2. Utility function (DRY: single formatting logic)
export function formatGarmentLabel(name: string): string { ... }

// 3. Context (Dependency Inversion: depend on context, not props)
export function useGarmentType(): GarmentType { ... }

// 4. Small, memoized component (SRP: only renders thumbnail)
export const ClothingThumbnail = memo(function ClothingThumbnail({ file }) {
  const type = useGarmentType();
  const config = THUMBNAIL_POSITIONS[type];
  return <Canvas camera={{ position: config.camera }}>...</Canvas>;
});

// 5. Container component (Composition: small, focused)
export const ClothingGrid = memo(function ClothingGrid({ items }) {
  return (
    <div className={styles.grid}>
      {items.map(item => (
        <div key={item.id}>
          <ClothingThumbnail file={item.file} />
          <p>{formatGarmentLabel(item.name)}</p>
        </div>
      ))}
    </div>
  );
});

// 6. Parent wraps with context (No prop drilling)
export function ClothingSection({ activeTab }) {
  return (
    <GarmentProvider garmentType={tabToType(activeTab)}>
      <ClothingGrid items={getItems(activeTab)} />
    </GarmentProvider>
  );
}
````

---

## 🎓 When to Apply

Apply these principles to **all new code** and **refactoring tasks**.

For quick fixes: Prefer to fix properly (split component, centralize config) over hacky patches.

---

**Last updated:** April 2026
