# Glossary

The words AtlasTint and its code use for one thing each.

| Term | Definition | Do not write |
| :--- | :--- | :--- |
| Preset | One map the app offers, such as World, Brazil or Spain, with its curated manifest, projection and progress policy. | map set, dataset |
| Manifest | A Preset's curated list of entities. It, not the raw geometry, decides what is selectable and what counts toward progress. | feature list |
| Entity | One selectable place in a Manifest, with a stable application-owned ID; a multi-polygon Entity is still one Entity. | feature, shape, polygon |
| Parent group | The group an Entity belongs to, such as a Brazilian region or a Spanish autonomous community; selecting it selects all its selectable children. | category |
| Selection | The set of selected Entity IDs with their order, selected time and optional visit date, which the app persists. | picks |
| Fill mode | How selected Entities are coloured. | color mode |
| Progress | The share of a Preset's primary Entities that are selected, against the total its Manifest defines. | score, count |
