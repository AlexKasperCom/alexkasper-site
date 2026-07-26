---
id:                   # Immutable Event ID, e.g. e000123. Replaced by new-event.
title:
slug:                 # Human-readable URL slug. Replaced by new-event from the title.

date: "{{ now.Format "2006-01-02" }}" # Date this record was created (auto-filled).

startDate:            # ISO 8601 with UTC offset, e.g. 2026-08-06T09:00:00-07:00
endDate:              # Optional. Same format as startDate. Defaults to startDate if omitted.

location:             # Venue name, e.g. "Las Vegas Convention Center"
city:
region:
country:

externalUrl:          # Official event website or registration link.

summary:              # One- or two-sentence summary.

tags:                 # Descriptive tags:
                      #   - conference
                      #   - defcon

projects:             # Related project slugs:
                      #   - manufactured-urgency

draft: true           # Set to false when ready to publish.
---

<!--

Notes:

This is an Event entry.

Write your own notes, observations, agenda, travel details, or post-event report here.

This content is not part of the event metadata.

Place optional representative image: cover.jpg in this leaf bundle folder.

-->
