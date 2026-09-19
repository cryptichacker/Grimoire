---
tags: [coding, python, scraping, html, beautifulsoup]
type: cheatsheet
source: compiled reference (BeautifulSoup 4)
last-verified: 2026-08-27
---

# BeautifulSoup

## Up
- [[Python]]

BeautifulSoup (bs4) parses HTML/XML into a navigable tree for scraping and extraction. Install: `pip install beautifulsoup4 lxml`. Often paired with [[Requests]] to fetch pages first.

---

## Getting Started

```python
import requests
from bs4 import BeautifulSoup

html = requests.get("https://example.com", timeout=10).text
soup = BeautifulSoup(html, "lxml")     # parser: "lxml" (fast), "html.parser" (stdlib), "html5lib" (lenient)

print(soup.prettify())                 # nicely indented tree
print(soup.title.text)                 # page <title>
```

| Parser | Notes |
|---|---|
| `html.parser` | Built in, no install, decent |
| `lxml` | Fast, needs `pip install lxml` (recommended) |
| `html5lib` | Most lenient, browser-like, slower |
| `lxml-xml` / `"xml"` | For XML documents |

---

## Finding Elements

```python
soup.find("a")                          # first <a>
soup.find("div", class_="content")      # class_ (trailing underscore!)
soup.find("input", id="email")
soup.find("a", attrs={"data-id": "5"})
soup.find(string="Exact text")

soup.find_all("a")                      # all <a> (a list)
soup.find_all("a", limit=5)
soup.find_all(["h1", "h2", "h3"])       # multiple tags
soup.find_all("p", class_="intro")
soup.find_all(href=True)                # tags that have an href attr
soup.find_all("a", string=lambda t: t and "next" in t.lower())
```

### CSS selectors (often cleanest)

```python
soup.select("div.content > p")          # list of matches
soup.select_one("#main .title")         # first match
soup.select("a[href^='https']")         # attribute starts-with
soup.select("ul li:nth-of-type(2)")
soup.select("table tr td")
```

| CSS pattern | Matches |
|---|---|
| `tag` | by tag name |
| `.cls` / `#id` | by class / id |
| `a > b` | direct child |
| `a b` | any descendant |
| `[attr=val]` `[attr^=x]` `[attr$=x]` `[attr*=x]` | attribute equals / starts / ends / contains |

---

## Extracting Data

```python
el = soup.find("a")
el.text                 # all text inside (recursively), incl. children
el.get_text(strip=True) # stripped text
el.get_text(", ")       # join descendant strings with separator
el.string               # text ONLY if it has a single child string

el["href"]              # attribute value (KeyError if missing)
el.get("href")          # None if missing
el.get("class")         # returns a LIST for class
el.attrs                # dict of all attributes
el.name                 # tag name, e.g. 'a'
```

```python
# collect all links
links = [a.get("href") for a in soup.find_all("a", href=True)]

# scrape a table
rows = []
for tr in soup.select("table tr"):
    cells = [td.get_text(strip=True) for td in tr.find_all(["td", "th"])]
    rows.append(cells)
```

---

## Navigating the Tree

```python
el.parent; el.parents               # up
el.contents; el.children            # down (direct)
el.descendants                      # down (all)
el.next_sibling; el.previous_sibling
el.find_next("p"); el.find_previous("h2")
el.find_parent("div")

# strip whitespace-only text nodes when navigating siblings
for sib in el.next_siblings:
    if sib.name:   # skip NavigableString newlines
        ...
```

---

## Modifying the Tree

```python
tag.string = "new text"
tag["class"] = "highlight"
tag.append("more")
new = soup.new_tag("span"); new.string = "hi"; tag.append(new)
tag.decompose()          # remove tag and its contents entirely
tag.extract()            # remove but return it
tag.unwrap()             # remove tag, keep its children
str(soup)                # serialize back to HTML
```

---

## End-to-End Example

```python
import requests
from bs4 import BeautifulSoup

def scrape_headlines(url):
    resp = requests.get(url, timeout=10,
                        headers={"User-Agent": "Mozilla/5.0 (scraper)"})
    resp.raise_for_status()
    soup = BeautifulSoup(resp.text, "lxml")
    items = []
    for art in soup.select("article"):
        title = art.select_one("h2")
        link = art.select_one("a[href]")
        if title and link:
            items.append({"title": title.get_text(strip=True),
                          "url": link["href"]})
    return items
```

---

## Tips & Etiquette

- Check `resp.status_code` / `raise_for_status()` before parsing.
- Elements can be `None` — guard before indexing (`if el:` then `el["href"]`).
- `class_` (with underscore) because `class` is a Python keyword; class values come back as a list.
- For JavaScript-rendered pages, bs4 sees only the initial HTML — use Selenium/Playwright to render first, then parse the result.
- Respect `robots.txt`, add a `User-Agent`, rate-limit, and cache responses when scraping.
- For big/streamed documents, `lxml`'s own `iterparse` or `parsel` can be more memory-efficient.
