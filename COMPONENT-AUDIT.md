# Mantto Component Audit

## Summary
**Total unique CSS classes:** ~300
**Potential for consolidation:** HIGH

---

## 🎴 CARD PATTERNS (7 variants → should be 1-2)

| Current Class | Used In | Notes |
|--------------|---------|-------|
| `.card` | Generic | Basic white card with shadow |
| `.stat-card` | Dashboard | Stats with icon + trend |
| `.prediction-card` | IA Predictiva | Has header/body/footer |
| `.asset-card` | Activos QR | QR code card |
| `.vendor-card` | Proveedores | Selectable vendor |
| `.team-card` | Equipo | Staff member card |
| `.tracking-card` | Seguimiento | Live tracking |
| `.config-card` | Configuración | Settings card |
| `.report-section-card` | Reportes | Report section |
| `.incident-card` | Floor Detail | Small incident item |
| `.order-card` | (via table rows) | Work order |

**RECOMMENDATION:** Create unified `.card` with modifiers:
- `.card--stat` 
- `.card--selectable` (hover/active states)
- `.card--compact`

---

## 🏷️ STATUS/BADGE PATTERNS (4+ variants → should be 1)

| Current | Colors | Used In |
|---------|--------|---------|
| `.status-badge` + `.status-dot` | urgent/pending/in-progress/resolved | Orders |
| `.prediction-badge` | high/medium | IA Predictiva |
| `.tech-list-badge` | available/working/in-transit | Equipo |
| `.timeline-days` | urgent/warning/ok | Cumplimiento |
| `.compliance-stat-icon` | urgent/ok | Cumplimiento |

**RECOMMENDATION:** Single `.badge` system:
```css
.badge { /* base */ }
.badge--urgent { background: var(--red-500); }
.badge--warning { background: var(--amber-500); }
.badge--success { background: var(--emerald-500); }
.badge--info { background: var(--indigo-500); }
.badge--neutral { background: var(--gray-500); }
```

---

## 📊 COLORED BOX PATTERNS (3 variants → should be 1)

| Current | Used In |
|---------|---------|
| `.cost-box.preventive` / `.cost-box.failure` | IA Predictiva costs |
| `.savings-box` | IA Predictiva savings |
| `.stat-card` with `.stat-icon.red/emerald/amber/indigo` | Dashboard |

**RECOMMENDATION:** Single `.highlight-box` with color modifiers:
```css
.highlight-box { padding: 16px; border-radius: var(--radius-md); text-align: center; }
.highlight-box--success { background: var(--emerald-500); color: #fff; }
.highlight-box--danger { background: var(--red-500); color: #fff; }
.highlight-box--warning { background: var(--amber-500); color: #fff; }
.highlight-box--info { background: var(--indigo-500); color: #fff; }
```

---

## 📋 GRID PATTERNS (5 variants → should be 2-3)

| Current | Columns | Gap |
|---------|---------|-----|
| `.stats-row` | 4 cols | 24px |
| `.dashboard-grid` | 2 cols | 24px |
| `.predictions-grid` | 3 cols | 24px |
| `.team-grid` | 4 cols | 20px |
| `.vendors-grid` | 3 cols | 20px |
| `.assets-grid` | 3 cols | 24px |
| `.config-grid` | 2 cols | 24px |
| `.report-grid` | 3 cols | 24px |

**RECOMMENDATION:** Generic grid utility:
```css
.grid { display: grid; gap: 24px; }
.grid--2 { grid-template-columns: repeat(2, 1fr); }
.grid--3 { grid-template-columns: repeat(3, 1fr); }
.grid--4 { grid-template-columns: repeat(4, 1fr); }
```

---

## 👤 AVATAR PATTERNS (5 variants → should be 1-2)

| Current | Size | Used In |
|---------|------|---------|
| `.team-avatar` | 64px | Equipo cards |
| `.tracking-avatar` | 48px | Seguimiento |
| `.tech-list-avatar` | 40px | Tech list |
| `.tech-avatar-sm` | 32px | Small contexts |
| `.user-avatar` | 32px | User card |
| `.assignee-avatar` | 28px | Order cards |

**RECOMMENDATION:** Single `.avatar` with size modifiers:
```css
.avatar { border-radius: 50%; object-fit: cover; }
.avatar--xs { width: 28px; height: 28px; }
.avatar--sm { width: 32px; height: 32px; }
.avatar--md { width: 40px; height: 40px; }
.avatar--lg { width: 48px; height: 48px; }
.avatar--xl { width: 64px; height: 64px; }
```

---

## 📜 TIMELINE PATTERNS (3 variants → should be 1)

| Current | Used In |
|---------|---------|
| `.activity-feed` + `.activity-item` | Dashboard |
| `.timeline-item` + `.timeline-content` | Compliance |
| `.drawer-timeline` | Order detail drawer |
| `.history-item` | Asset history |

**RECOMMENDATION:** Single `.timeline` system with variants for vertical line position.

---

## 🔘 BUTTON PATTERNS (OK - already unified)

`.btn`, `.btn-primary`, `.btn-secondary` — **GOOD**

Only addition needed: `.btn--sm` for smaller contexts.

---

## 📝 FORM PATTERNS (OK - already unified)

`.form-group`, `.form-label`, `.form-input`, `.form-select` — **GOOD**

---

## 🎨 SECTION HEADER PATTERNS (4 variants → should be 1)

| Current | Used In |
|---------|---------|
| `.card-header` + `.card-title` | Generic cards |
| `.config-card-header` + `.config-card-title` | Config |
| `.report-header` + `.report-title` | Reports |
| `.tracking-header` | Tracking |
| `.team-header` | Team |

**RECOMMENDATION:** Unify into `.section-header` with `.section-title`.

---

## 🚦 PRIORITY ACTIONS

### Phase 1: Quick Wins (30 min)
1. Unify badge/status colors into single system
2. Unify colored boxes (cost/savings)
3. Create grid utility classes

### Phase 2: Medium Effort (1-2 hrs)
4. Consolidate card variants
5. Unify avatar sizes
6. Standardize section headers

### Phase 3: Deeper Refactor (2-3 hrs)
7. Consolidate timeline patterns
8. Create component documentation

---

## 🎯 Target: Reduce CSS by ~40%

Current estimate: ~1500 lines of CSS
Target: ~900 lines with reusable utilities
