---
name: playwright
description: Driving Chromium from Python with Playwright — printing HTML to PDF, opening a local file, and keeping the browser out of the tested part of the code. Use when generating a PDF from HTML, automating a browser check of a rendered page, or deciding where the boundary between a pure renderer and a browser belongs.
---

# Playwright, from Python

```bash
pip install playwright
python -m playwright install chromium     # ~150 MB of browser, once per machine
```

The browser is a separate download from the package. A machine with the package
and no browser fails at `launch()`, not at import, which is why the check below
is worth having.

## Printing HTML to PDF

**PDF generation is Chromium only.** Firefox and WebKit do not implement it.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as playwright:
    browser = playwright.chromium.launch()      # headless by default
    page = browser.new_page()
    page.goto(html_path.as_uri())               # file:///C:/... — as_uri(), never a string
    page.pdf(
        path=str(pdf_path),
        format="A4",
        print_background=True,                  # off by default: colours and shading vanish
        margin={"top": "18mm", "bottom": "18mm", "left": "16mm", "right": "16mm"},
    )
    browser.close()
```

`page.pdf` also takes `width`/`height` instead of `format`,
`display_header_footer` with `header_template`/`footer_template` (whose
`class="pageNumber"`, `"totalPages"`, `"title"`, `"date"`, `"url"` spans are
filled in), and `prefer_css_page_size`.

Two things that are easy to get wrong:

- **`print_background=True` or the page prints white.** The default drops
  background colours and images, so a cover that looked right in the browser
  comes out blank.
- **Use `Path.as_uri()`**, not `"file://" + str(path)`. On Windows the second
  produces `file://C:\...`, which Chromium reads as a host named `C`.

For HTML held in memory, `page.set_content(html)` avoids the temporary file —
but relative links to CSS, fonts or images then resolve against nothing. Write
the file and `goto` it when the document refers to anything beside it.

## Keep the browser out of the part you test

A renderer that returns a string is testable everywhere and costs nothing; a
renderer that launches a browser is testable only where a browser is installed.
Split them:

```python
def render(run) -> str:
    """HTML from the archive. Pure: no browser, no filesystem, no clock."""
    ...

def print_to_pdf(html_path: Path, pdf_path: Path) -> None:
    """The browser half. Imported here, not at module scope, so the module
    loads on a machine that has no browser and the tests still run."""
    from playwright.sync_api import sync_playwright
    ...
```

The import inside the function is deliberate. At module scope it makes every
importer depend on the package; inside, the failure arrives at the one call that
genuinely needs it, and with a message you control:

```python
try:
    from playwright.sync_api import sync_playwright
except ImportError as exc:
    raise RuntimeError(
        "playwright is not installed: pip install playwright && "
        "python -m playwright install chromium"
    ) from exc
```

That split is the same rule as *code before agent*: the structure that can be
checked cheaply is checked cheaply, and the part that needs a real dependency is
demonstrated rather than tested — class **D**, and worth saying so.

## Automating a check of a rendered page

```python
page.goto(html_path.as_uri())
assert page.locator("nav a").count() == chapters           # the index has an entry per chapter
assert page.locator("#cover .dedication").is_visible()
page.screenshot(path="docs/diagrams/reader.png", full_page=True)
```

A screenshot is evidence a reader can look at; an assertion is evidence a test
can keep. A documented browser session is the honest middle when the thing being
checked is whether a human would see it — record what was opened, what was
looked at, and what was seen.

## Headless, and one limitation

`launch()` is headless; `launch(headless=False)` opens a window. Headless
Chromium **cannot navigate to an existing PDF document** (an upstream Chromium
limitation) — which does not affect generating one, only opening one afterwards.
To check a PDF, check the HTML it was printed from, or open the PDF with
something else.
