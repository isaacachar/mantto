# Mantto Color Palette Audit

## Official Palette (CSS Variables)

### Primary (Indigo)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--indigo-50` | #eef2ff | Light backgrounds |
| `--indigo-100` | #e0e7ff | Hover states |
| `--indigo-500` | #6366f1 | Secondary accent |
| `--indigo-600` | #4f46e5 | **Primary brand** ⭐ |
| `--indigo-700` | #4338ca | Hover on primary |
| `--indigo-900` | #1e1b4b | Sidebar background |

### Success (Emerald)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--emerald-50` | #ecfdf5 | Light success bg |
| `--emerald-500` | #10b981 | **Success solid** ⭐ |
| `--emerald-600` | #059669 | Success text |

### Warning (Amber)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--amber-50` | #fffbeb | Light warning bg |
| `--amber-500` | #f59e0b | **Warning solid** ⭐ |
| `--amber-600` | #d97706 | Warning text |

### Danger (Red)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--red-50` | #fef2f2 | Light danger bg |
| `--red-500` | #ef4444 | **Danger solid** ⭐ |
| `--red-600` | #dc2626 | Danger text |

### Info (Blue)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--blue-50` | #eff6ff | Light info bg |
| `--blue-500` | #3b82f6 | Info accent |
| `--blue-600` | #2563eb | Info text |

### Accent (Purple)
| Variable | Hex | Usage |
|----------|-----|-------|
| `--purple-50` | #faf5ff | Light purple bg |
| `--purple-500` | #8b5cf6 | **AI/Insights** ⭐ |
| `--purple-600` | #7c3aed | Purple text |

### Neutrals
| Variable | Hex | Usage |
|----------|-----|-------|
| `--bg-primary` | #ffffff | Card backgrounds |
| `--bg-secondary` | #f8fafc | Page background |
| `--bg-tertiary` | #f1f5f9 | Subtle backgrounds |
| `--text-primary` | #0f172a | Headings, main text |
| `--text-secondary` | #475569 | Body text |
| `--text-muted` | #94a3b8 | Captions, hints |
| `--border` | #e2e8f0 | Standard borders |
| `--border-light` | #f1f5f9 | Subtle dividers |

---

## ⚠️ Hardcoded Colors (Should Use Variables)

### WhatsApp Theme (Acceptable - Brand Colors)
| Color | Used For |
|-------|----------|
| `#25D366` | WhatsApp green |
| `#075e54` | WhatsApp dark green |
| `#128c7e` | WhatsApp teal |
| `#ece5dd` | WhatsApp chat bg |
| `#dcf8c6` | WhatsApp sent bubble |
| `#667781` | WhatsApp text |

**Verdict:** ✅ Keep as-is (brand-specific)

### Hardcoded Duplicates (Should Be Variables)
| Hardcoded | Should Be |
|-----------|-----------|
| `#fff` (25x) | `var(--bg-primary)` or keep for text |
| `#000` (8x) | QR pattern only - OK |
| `#f1f5f9` (2x) | `var(--border-light)` |

### One-Off Colors (Review Needed)
| Color | Location | Recommendation |
|-------|----------|----------------|
| `#fef9c3` | Floor row selected (line 340) | Add `--amber-100` |
| `#fef3c7` | Floor bar level-2 (line 348) | Add `--amber-100` |
| `#fed7aa` | Floor bar level-3 (line 349) | Add `--amber-200` |
| `#fecaca` | Floor bar level-4 (line 350) | Add `--red-100` |
| `#fcd34d` | Warning banner border (line 2469) | Add `--amber-300` |
| `#d1fae5` | Floor bar level-1 (line 347) | Add `--emerald-100` |
| `#dcfce7` | Icon background (line 2862) | Add `--emerald-100` |

### Missing from Palette (Add These)
```css
--slate-100: #f1f5f9;  /* Used but not defined */
--slate-600: #475569;  /* Used but not defined */
--indigo-300: #a5b4fc; /* Used for hover states */
--emerald-100: #d1fae5;
--emerald-700: #047857;
--amber-100: #fef3c7;
--amber-700: #b45309;
--amber-800: #92400e;
--red-100: #fee2e2;
```

---

## 📊 Usage Summary

| Category | Count | Status |
|----------|-------|--------|
| CSS Variables | 450+ refs | ✅ Good |
| `#fff` / `#000` | 33 | ⚠️ Mostly OK |
| WhatsApp colors | 6 | ✅ Brand-specific |
| One-off colors | ~10 | ❌ Clean up |

---

## 🎯 Recommendations

### Priority 1: Add Missing Variables
Add these to `:root`:
```css
--emerald-100: #d1fae5;
--emerald-700: #047857;
--amber-100: #fef3c7;
--amber-700: #b45309;
--red-100: #fee2e2;
--indigo-300: #a5b4fc;
```

### Priority 2: Replace One-Offs
Search and replace hardcoded colors with variables.

### Priority 3: Document Color Usage
Create a style guide showing when to use each color.

---

## ✅ Overall Assessment

**Score: 8/10** — Palette is well-structured with CSS variables. Minor cleanup needed for one-off colors and missing intermediate shades.

**Consistency:** High — Most colors use the variable system.
**Brand Alignment:** Good — Indigo primary, Emerald success, Purple for AI.
