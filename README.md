# Maine news articles

Every article the [Maine News Ecosystem Map](https://maine-news-map.onrender.com) is built
from: what Media Cloud records about it, the places and topics the map tags it with, and
the full text.

**Private, for research only.** The article text belongs to the outlets that published it.
Please don't share these files outside the project or republish the text.

The map's daily update writes this repo around noon Eastern, with `python main.py news
archive` from [maine-news-mapping](https://github.com/AshishJaiswal25/maine-news-mapping).
This README is written by it too, so edits here are overwritten.

## What's here

| | |
|---|---|
| Articles | 14,532 |
| With text | 13,522 |
| Distinct stories, copies merged | 10,606 |
| Outlets | 27 |
| Days | Sep 30, 2026 to Nov 5, 2026 (37 days) |
| Last updated | Nov 6, 2026 |

- `articles/YYYY-MM-DD.jsonl`: the articles published that day, one JSON object per line,
  text included.
- `metadata/YYYY-MM.csv`: the same articles without the text, one file per month, for
  Excel. (An Excel cell holds at most 32,767 characters, too few for some articles.)

Loading every day in Python:

```python
from pathlib import Path
import pandas as pd

articles = pd.concat(
    pd.read_json(path, lines=True, dtype={"id": str, "story_id": str})
    for path in sorted(Path("articles").glob("*.jsonl"))
)
```

## Where it comes from

- **Articles:** Media Cloud's Maine collection (id 38381370):
  each story it indexed from a Maine outlet, one row per outlet's copy. Outlets Media
  Cloud doesn't index aren't here.
- **Text:** fetched from each article's own page and extracted with trafilatura. Pages
  whose robots.txt disallows it were not fetched. A paywalled page often gives only a
  teaser.
- **Places and topics:** tagged by Claude (claude-sonnet-5-5) from the headline
  and text, with each place checked against the Census list of Maine towns and counties.
  They are the map's tags, not the outlets'.
- **Copies:** a story that several outlets ran, or one outlet posted twice, is merged into
  one story. Every copy is kept here, linked by `story_id`; the copy marked `canonical`
  is the one the map lists.

The map keeps a rolling 90-day window. A day here is updated
while it is in the window, so late tags reach it, and kept as it was once the window has
moved on, so the archive keeps growing.

## Fields

| Field | What it is |
|---|---|
| `id` | Media Cloud's id for this copy |
| `story_id` | The merged story this copy belongs to; copies of one story share it |
| `dedup_reason` | `canonical` (the copy the map shows), or why it was merged: `title` (same headline), `text` (same body under another headline) |
| `published_at` | Publication date (sometimes with a time), as Media Cloud gives it |
| `outlet` | The outlet's web domain |
| `outlet_home_town`, `outlet_home_county` | Where the outlet is based, when known |
| `outlet_reach` | `local` or `statewide`: whether the outlet covers a locality or the state |
| `title`, `url`, `language` | As Media Cloud gives them |
| `scope` | What the story is about: `local` (one community), `regional` (several nearby, or a county), `statewide` (Maine as a whole), `national` (not specifically about Maine), or `not_maine` (somewhere else entirely) |
| `places` | Maine towns and counties the story is tagged to; `role` is `primary` (what it is about) or `mentioned` |
| `main_topic`, `also_topics` | The story's main topic, and any others it also covers |
| `geotag_status`, `topic_status` | `ok` once tagged; `not_tagged` or `not_classified` before the day's tagging has run |
| `text_status` | `ok`; `short` (under 600 characters, usually a paywall teaser); `robots` (robots.txt disallows fetching, not fetched); `http_error` (the page returned an error); `failed` (the request failed); `not_fetched` (not tried yet) |
| `text_chars`, `text_fetched_at` | The text's length, and when it was fetched |
| `text` | The article text, or `null` when there is none |

The CSVs have the same rows, with `places` split into `primary_places`, `mentioned_places`
and `counties`, and without the text.
