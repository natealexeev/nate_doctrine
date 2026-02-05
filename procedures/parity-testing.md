# Parity Testing Procedure

> Visual comparison testing between two implementations (e.g., React vs Vue, old vs new).

---

## Overview

Parity testing verifies that two implementations produce visually identical results. This is critical for:
- Framework migrations (React → Vue, Angular → React)
- Major refactors
- Design system updates
- Component library swaps

---

## Prerequisites

- Two running instances (reference and test)
- Browser testing agents set up (see [browser-testing.md](browser-testing.md))
- **ImageMagick** installed for screenshot comparison

```bash
# Install ImageMagick
# Debian/Ubuntu
sudo apt install imagemagick

# macOS
brew install imagemagick

# Arch
sudo pacman -S imagemagick

# Verify installation
magick -version
```

---

## The Procedure

### 1. Prepare Both Instances

```bash
# Reference instance (e.g., production React app)
REFERENCE_URL="http://localhost:3000"

# Test instance (e.g., new Vue implementation)
TEST_URL="http://localhost:3011"
```

Ensure both instances:
- Are running and accessible
- Have the same test data
- Are in the same application state (logged in, same user, etc.)

### 2. Define Test Matrix

Create a list of pages/states to compare:

```
/dashboard
/users
/users/123
/settings
/settings (dark mode)
/reports?period=monthly
```

Include:
- All major pages
- Different viewport sizes (desktop, tablet, mobile)
- Different states (empty, loading, populated, error)
- Theme variations (light/dark)

### 3. Capture Screenshots

For each page in the test matrix:

```bash
# Using MCP tools (via Claude)
# 1. Navigate reference browser to page
# 2. Wait for network idle
# 3. Take screenshot: reference-{page}.png
# 4. Navigate test browser to same page
# 5. Wait for network idle
# 6. Take screenshot: test-{page}.png
```

**Critical:** Use identical viewport sizes for both browsers.

### 4. Compare with ImageMagick

```bash
# Basic comparison - creates diff image
magick compare reference.png test.png diff.png

# Comparison with metrics (returns difference percentage)
magick compare -metric AE reference.png test.png diff.png 2>&1

# More sensitive comparison (highlight small differences)
magick compare -metric RMSE reference.png test.png diff.png 2>&1

# Comparison with fuzz factor (ignore minor color differences)
magick compare -fuzz 5% -metric AE reference.png test.png diff.png 2>&1
```

### 5. Interpret Results

| Metric | Meaning |
|--------|---------|
| `AE` (Absolute Error) | Number of different pixels. 0 = identical. |
| `RMSE` (Root Mean Square Error) | Average difference. Lower = more similar. |
| `PSNR` (Peak Signal to Noise Ratio) | Higher = more similar. Inf = identical. |

**Diff image colors:**
- **Red pixels** = Present in reference, missing in test
- **White pixels** = Identical
- **Other colors** = Differences

---

## Batch Comparison Script

```bash
#!/bin/bash
# compare-all.sh

REFERENCE_DIR="./screenshots/reference"
TEST_DIR="./screenshots/test"
DIFF_DIR="./screenshots/diff"
REPORT="./parity-report.txt"

mkdir -p $DIFF_DIR
echo "Parity Test Report - $(date)" > $REPORT
echo "================================" >> $REPORT

for ref in $REFERENCE_DIR/*.png; do
  filename=$(basename "$ref")
  test="$TEST_DIR/$filename"
  diff="$DIFF_DIR/$filename"

  if [ -f "$test" ]; then
    # Compare and capture metric
    result=$(magick compare -metric AE "$ref" "$test" "$diff" 2>&1)

    if [ "$result" = "0" ]; then
      echo "✓ PASS: $filename (identical)" >> $REPORT
    else
      echo "✗ FAIL: $filename ($result pixels differ)" >> $REPORT
    fi
  else
    echo "? SKIP: $filename (no test screenshot)" >> $REPORT
  fi
done

echo "" >> $REPORT
echo "Diff images saved to: $DIFF_DIR" >> $REPORT

cat $REPORT
```

---

## Handling Acceptable Differences

Some differences are expected and acceptable:

### Timestamps/Dates
- Mock the date in both instances, or
- Mask date regions before comparison:

```bash
# Mask a region (x, y, width, height)
magick reference.png -region 200x30+100+50 -fill gray reference-masked.png
magick test.png -region 200x30+100+50 -fill gray test-masked.png
magick compare reference-masked.png test-masked.png diff.png
```

### Random Content (avatars, IDs)
- Use seeded random data
- Or mask those regions

### Animation Frames
- Disable animations in test mode
- Or wait for animation completion

### Font Rendering
- Minor anti-aliasing differences are normal across systems
- Use fuzz factor: `magick compare -fuzz 2% ...`

---

## Automated Parity Pipeline

```bash
#!/bin/bash
# parity-test.sh

set -e

PAGES=(
  "/"
  "/dashboard"
  "/users"
  "/settings"
)

REFERENCE_PORT=3000
TEST_PORT=3011
VIEWPORT="1920x1080"

# Capture all screenshots
for page in "${PAGES[@]}"; do
  safe_name=$(echo "$page" | tr '/' '_')

  # Reference screenshot (using your MCP tool or puppeteer/playwright)
  capture_screenshot "http://localhost:$REFERENCE_PORT$page" \
    "reference$safe_name.png" "$VIEWPORT"

  # Test screenshot
  capture_screenshot "http://localhost:$TEST_PORT$page" \
    "test$safe_name.png" "$VIEWPORT"
done

# Compare all
./compare-all.sh

# Fail if any differences
if grep -q "FAIL" parity-report.txt; then
  echo "Parity test FAILED - see diff images"
  exit 1
fi

echo "Parity test PASSED"
```

---

## DO's and DON'Ts

### DO

- **Use ImageMagick** for comparison - it's the standard tool for this
- **Capture at identical viewport sizes** - even 1px difference breaks comparison
- **Wait for network idle** - no spinners or loading states
- **Disable animations** - or wait for completion
- **Test all states** - empty, loading, populated, error
- **Document acceptable differences** - with justification
- **Version control your reference screenshots** - they're your source of truth

### DON'T

- **Don't compare against live/production** - too many variables
- **Don't ignore "minor" differences** - they compound
- **Don't skip mobile viewports** - responsive bugs are common
- **Don't forget dark mode** - if applicable
- **Don't compare without stable data** - mock your APIs
- **Don't trust "looks close enough"** - use metrics

---

## When Parity Fails

1. **Examine the diff image** - Red areas show what's different
2. **Check the obvious first** - Viewport size, scroll position, loading state
3. **Isolate the component** - Is it a specific component or layout issue?
4. **Check CSS** - Most parity failures are CSS differences
5. **Document and decide** - Is this a bug or acceptable variation?

---

## Summary

```
1. Set up reference and test instances
2. Define test matrix (pages, states, viewports)
3. Capture screenshots from both
4. Compare with ImageMagick: magick compare -metric AE ref.png test.png diff.png
5. Review diff images and metrics
6. Fix failures or document acceptable differences
7. Repeat until 0 differences
```

**Remember:** Parity testing is about catching visual regressions before users do. A 1-pixel difference today becomes a 10-pixel difference tomorrow.
