# Pension Calculator Audit Findings

Audit date: 2026-05-07

## Critical

### C1. No Subresource Integrity (SRI) on CDN Script
- **Line:** 7
- **Risk:** CDN compromise could inject arbitrary JS with full DOM access
- **Fix:** Pin Chart.js version and add `integrity` + `crossorigin` attributes

### C2. Hardcoded Current Age (30)
- **Line:** 779, 1059
- **Risk:** Every user who is not 30 gets wrong projections
- **Fix:** Add "Current Age" slider (18–65) to sidebar, wire into state

### C3. Retirement Age <= Current Age Crashes
- **Line:** 839
- **Risk:** `totalMonths` becomes 0 or negative, `salaryData[length-1]` breaks
- **Fix:** Guard with validation, clamp retirement age slider min to current age + 1

### C4. Salary Cap Off-by-One
- **Line:** 866
- **Risk:** `yearsElapsed > capYears` should be `>=`, cap engages one year late
- **Fix:** Change to `>=`

## Important

### I1. Zero Accessibility
- No `aria-label` on sliders
- Tab bar lacks `role="tablist"`, `role="tab"`, `aria-selected`, `aria-controls`
- Summary cards have no `aria-live` regions
- `.text-input:focus` removes outline with no replacement (WCAG 2.4.7)
- Modal has no focus trap, `role="dialog"`, `aria-modal`, escape-to-close, or focus return
- Slider thumb focus styles suppressed by custom styling
- Summary card label opacity may fail WCAG AA contrast

### I2. No Debounce on Slider Input
- **Lines:** 1382–1397
- **Risk:** ~60 recalculations/sec during drag, layout thrashing
- **Fix:** `requestAnimationFrame` debounce wrapper

### I3. NIC Saving Ignores Secondary Threshold
- **Line:** 821
- **Risk:** Overstates NIC saving at low salaries
- **Fix:** Calculate as difference in employer NIC before/after sacrifice

### I4. CSV Download Missing Projection Data
- **Lines:** 1474–1504
- **Risk:** Users expect to download the projection, not just reference data
- **Fix:** Include year-by-year projection + input parameters in CSV

### I5. Hidden Tab Chart Sizing Bug
- **Lines:** 1091–1094
- **Risk:** Real Value chart gets wrong dimensions until window resize
- **Fix:** Call `chart.resize()` on tab switch, or lazy-create charts

### I6. Mobile Layout Overflow
- **Lines:** 379–387
- **Risk:** Charts too tall, income tab overflows on narrow screens
- **Fix:** Add 480px breakpoint, stack income cards, reduce chart height

## Suggestions

### S1. Year Rounding Bug
- **Line:** 855
- `Math.round(currentYear + month/12)` produces wrong calendar year for some months
- **Fix:** Use `currentYear + Math.floor(month / 12)`

### S2. Dead Code: `deriveInflationDefault()`
- Defined but inflation default is hardcoded to 0.025
- **Fix:** Remove or repurpose with post-1997 filter

### S3. No URL State Persistence
- All inputs reset on reload, can't share scenarios
- **Fix:** Serialize state to URL search params

### S4. Tab Buttons Missing `type="button"`
- Defensive measure against accidental form submission

### S5. No `<noscript>` Fallback
- Page blank without JS

### S6. Modal Close Button
- Uses plain `x` character, needs `aria-label="Close"` and proper icon

### S7. `formatCurrency` Precision Jump
- Full precision up to 99,999, then jumps to "Xk" at 100k
- **Fix:** Keep full formatting to 999,999, abbreviate only at 1M+

### S8. Self-Tests Run in Production
- Console noise on every page load
- **Fix:** Gate behind `?debug` URL parameter
