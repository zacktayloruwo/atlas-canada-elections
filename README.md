# Atlas of Canadian Elections

Federal and provincial general elections in Canada since 1867: every contest and every candidate, mapped on the district boundaries of the day, with district lineages across redistributions. Any map can also be drawn as a Dorling cartogram: one circle per seat, its area the electors per member (Federal mode: population per member); a district without a count takes the median of the recorded ones and is outlined dashed. The site has two modes, Provincial and Federal, chosen in its top bar.

**Site:** https://zacktayloruwo.github.io/atlas-canada-elections/

This repository holds the published site: a static web app (Svelte, DuckDB-WASM) and the data it reads. It is written by the publish step of the Provincial Elections Dataset (PED) pipeline; nothing is edited here by hand. Each publish replaces the repository's history with a single commit, so only the current build is kept. Build `20261106T193703Z`, pipeline commit `0b03c0a`, built 2026-11-06T19:37:03Z.

## Using the data

Everything the site shows is in `data/` as Parquet files, readable with DuckDB, Arrow, pandas, R (`arrow`, `nanoparquet`) or any Parquet tool, directly from this repository or from the site URL. The provincial tables are in `data/`; the federal tables, with the same names and columns, are in `data/federal/`.

```sql
-- DuckDB
SELECT prabbr, eyear, count(*) AS contests, sum(magnitude) AS seats
FROM 'https://zacktayloruwo.github.io/atlas-canada-elections/data/districts.parquet'
GROUP BY 1, 2 ORDER BY 1, 2;
```

Identifiers: `pedid` is the contest key (province + election year + a sequence, plus a ballot letter where a district returned members on separate ballots). It is not a longitudinal key; `lineage_id` (from `lineages.parquet`) follows a district's territory across boundary changes. Candidates carry no person identifier: rows are grouped by a normalized name within a province (`person_key`, regenerated on every build), and `link_basis` says how each row was attached.

Vote measures are never mixed: `votes_first` and `share_first` are first counts; `votes_final` and `share_final` exist only for alternative-vote and single-transferable-vote contests, and shares only where every candidate has a final count. `quality` is the confidence rating of each contest and candidate, A to D; `dq` in `source_use` is the data point quality of a source (DQ, 1 to 5, 5 best). `data/manifest.json` carries the definitions of every derived measure.

`franchise.parquet` marks, for each province, when women were enfranchised, when the property or taxpayer qualification ended and when status Indians got the vote (from the researcher's compilation of franchise expansions, with the Act, source and confidence of each row). `governments.parquet` lists every first minister and government since 1867, one row per first minister per legislature (party in power as printed and as a family, dates, the general election that produced the legislature, seats won, majority or minority status with its basis, sources and verification), compiled from legislature and elections-agency sites corroborated with Wikipedia and other references; `elections` carries each election's polling date and the government it produced (`gov_*`) from the same compilation.

### Provincial tables (`data/`)

| File | Rows | Columns |
|---|---|---|
| `provinces.parquet` | 10 | prabbr, name_en, name_fr, proj4, snap_tol_m, first_eyear, last_eyear, n_elections |
| `elections.parquet` | 382 | prabbr, eyear, election_date, structure, ballots, formula, n_dist_expected, n_seats_expected, authority, n_dist_returned, n_contests, n_seats_returned, ro_year, has_geometry, label_scheme, gov_raw, gov_family, gov_first_minister, gov_status |
| `districts.parquet` | 21,380 | pedid, prabbr, eyear, pedname_en, pedname_fr, district_base, ballot, geo_role, geo_id, lineage_id, n_contests_on_polygon, magnitude, n_candidates, formula, contested, acclaimed, unfilled, outcome_kind, outcome_note_en, outcome_note_fr, outcome_filled_by, votes_total, votes_total_final, electorate, turnout_pct, margin_first, margin_final, margin_measure, winner_family, seat_split, quality, n_sources |
| `candidates.parquet` | 76,406 | pedid, prabbr, eyear, cluster_id, name, name_key, person_key, link_basis, party_raw, party_source, family, family2, votes_first, votes_final, votes_final_source, share_first, share_final, rank_first, rank_final, elected, quality, slug |
| `seats.parquet` | 23,165 | pedid, prabbr, eyear, cluster_id, seat_index, family, family2 |
| `parties.parquet` | 301 | family, label_en, label_fr, colour_light, colour_dark, pattern, sort_order, n_rows, n_seats, tier |
| `party_labels.parquet` | 927 | prabbr, party_raw, family, family2, first_eyear, last_eyear, n, sources |
| `vacancies.parquet` | 7 | prabbr, eyear, district_name, ballot, outcome_kind, reason, reason_fr, filled_by, citation |
| `sources.parquet` | 45 | src_abbr, title, publisher, years, provinces, type, url, status, rank, note, citation_en, citation_fr |
| `search_index.parquet` | 104,683 | type, prabbr, display_en, display_fr, key_norm, key_words, route |
| `geo_index.parquet` | 10,072 | geo_id, prabbr, ro_year, pednum, pedname, geo_role, bbox_xmin, bbox_ymin, bbox_xmax, bbox_ymax, centroid_x, centroid_y, ov_centroid_x, ov_centroid_y, areakm_net |
| `geo_join.parquet` | 20,889 | pedid, prabbr, eyear, geo_id, geo_role, join_basis |
| `geo_orphans.parquet` | 488 | side, prabbr, eyear, ro_year, name, id, reason |
| `era_crosswalk.parquet` | 15,531 | prabbr, ro_from, ro_to, geo_id_from, geo_id_to, area_km2, frac_of_from, frac_of_to, same_lineage |
| `geo_neighbours.parquet` | 22,810 | prabbr, ro_year, geo_id_a, geo_id_b |
| `lineages.parquet` | 260 | lineage_id, prabbr, first_eyear, last_eyear, n_contests, n_elections, n_branches_max, names_en, names_fr, lineage_basis, has_floterial, n_eras, n_polygons |
| `source_use.parquet` | 2,032 | prabbr, eyear, field, src_abbr, dq, n |
| `geo_unrepresented.parquet` | 21 | geo_id, prabbr, ro_year, eyear_min, eyear_max, area_km2, basis_en, basis_fr, citation |
| `governments.parquet` | 613 | prabbr, seq, first_minister, fm_number, party_raw, family, family2, coalition, start_date, end_date, start_reason, legislature, election_date, eyear, seats_won, total_seats, status, status_basis, pre_confederation, verification, sources, source_urls, notes |
| `franchise.parquet` | 27 | prabbr, event, date, date_precision, eyear, group_affected, description, legal_instrument, citation, url, confidence, note |
| `geo_cities.parquet` | 1,794 | geo_id, prabbr, ro_year, city_key, city_en, city_fr, share, census_year, basis |

`data/geo/` holds one TopoJSON per province at full resolution in that province's conic projection (`<PR>.topo.json`, one object per boundary era, arcs shared across eras), an overview version in EPSG:3347 (`<PR>.overview.topo.json`), `canada.topo.json` with the province outlines, and `provinces.periods.topo.json` with the province and territory outlines by period (one object per period in which no provincial boundary changed: `p1859` … `p1920` from the Historical Atlas of Canada's Territorial Evolution on the 2021 coastline, `p1927` and `p1999` from Statistics Canada 2021; each feature carries `from` and `to`, the years it applies to). Feature ids are `geo_id` values that `geo_join.parquet` links to contests.

### Federal tables (`data/federal/`)

Build `20261106T115418Z`. Every general election to the House of Commons since 1867, in every province and territory. `pedid` is `F<election year>_<distid>`, where `distid` is the source's district identifier (representation order year + province code + district number), kept as `district_base`. Careers are national: a candidate's appearances are linked by the source's own candidate identifier where it has one (`link_basis` `candid`), otherwise by name within a province (`name`, or `name+candid` where the name joins exactly one identified career); the identifier itself is not published. Every data point has DQ 5 and every contest the confidence rating A: the data are shown as published. Federal-only columns: `districts.region`, `districts.ro_year` (the contest's own representation order), `districts.contest_date`; `candidates.name_source` (the name as printed), `surname`, `incumbent`; and the table `general_elections` (one row per general election, with the party that formed the government). `governments.parquet` holds the federal governments and prime ministers (jurisdiction code `CA`), as the provincial one does for the provinces.

| File | Rows | Columns |
|---|---|---|
| `provinces.parquet` | 13 | prabbr, name_en, name_fr, proj4, snap_tol_m, first_eyear, last_eyear, n_elections |
| `elections.parquet` | 480 | prabbr, eyear, election_date, structure, ballots, formula, n_dist_expected, n_seats_expected, authority, n_dist_returned, n_contests, n_seats_returned, ro_year, has_geometry, label_scheme, gov_raw, gov_family, gov_first_minister, gov_status |
| `general_elections.parquet` | 45 | eyear, election_date, ro_year, ro_years, n_contests, n_seats, n_jurisdictions, gov_raw, gov_family |
| `districts.parquet` | 11,604 | pedid, prabbr, eyear, pedname_en, pedname_fr, district_base, ballot, geo_role, geo_id, lineage_id, n_contests_on_polygon, magnitude, n_candidates, formula, contested, acclaimed, unfilled, outcome_kind, outcome_note_en, outcome_note_fr, outcome_filled_by, votes_total, votes_total_final, electorate, turnout_pct, margin_first, margin_final, margin_measure, winner_family, seat_split, quality, n_sources, region, ro_year, contest_date, population, prabbrs, jurisdiction_note_en, jurisdiction_note_fr |
| `candidates.parquet` | 46,088 | pedid, prabbr, eyear, cluster_id, name, name_key, person_key, link_basis, party_raw, party_source, family, family2, votes_first, votes_final, votes_final_source, share_first, share_final, rank_first, rank_final, elected, quality, slug, name_source, surname, incumbent |
| `seats.parquet` | 11,726 | pedid, prabbr, eyear, cluster_id, seat_index, family, family2 |
| `parties.parquet` | 116 | family, label_en, label_fr, colour_light, colour_dark, pattern, sort_order, n_rows, n_seats, tier |
| `party_labels.parquet` | 616 | prabbr, party_raw, family, family2, first_eyear, last_eyear, n, sources |
| `source_use.parquet` | 2,282 | prabbr, eyear, field, src_abbr, dq, n |
| `vacancies.parquet` | 0 | prabbr, eyear, district_name, ballot, outcome_kind, reason, reason_fr, filled_by, citation |
| `sources.parquet` | 4 | src_abbr, title, publisher, years, provinces, type, url, status, rank, note, citation_en, citation_fr |
| `search_index.parquet` | 59,429 | type, prabbr, display_en, display_fr, key_norm, key_words, route |
| `geo_index.parquet` | 4,888 | geo_id, prabbr, ro_year, pednum, pedname, geo_role, bbox_xmin, bbox_ymin, bbox_xmax, bbox_ymax, centroid_x, centroid_y, ov_centroid_x, ov_centroid_y, areakm_net |
| `geo_join.parquet` | 11,604 | pedid, prabbr, eyear, geo_id, geo_role, join_basis |
| `geo_orphans.parquet` | 5 | side, prabbr, eyear, ro_year, name, id, reason |
| `era_crosswalk.parquet` | 7,908 | prabbr, ro_from, ro_to, geo_id_from, geo_id_to, area_km2, frac_of_from, frac_of_to, prabbr_to, same_lineage |
| `geo_neighbours.parquet` | 11,079 | prabbr, ro_year, geo_id_a, geo_id_b |
| `lineages.parquet` | 97 | lineage_id, prabbr, first_eyear, last_eyear, n_contests, n_elections, n_branches_max, names_en, names_fr, lineage_basis, has_floterial, n_eras, n_polygons, prabbrs |
| `governments.parquet` | 60 | prabbr, seq, first_minister, fm_number, party_raw, family, family2, coalition, start_date, end_date, start_reason, legislature, election_date, eyear, seats_won, total_seats, status, status_basis, pre_confederation, verification, sources, source_urls, notes |

`data/federal/geo/` holds one TopoJSON per province or territory (`<PR>.topo.json`, one object per representation order, in the jurisdiction's projection), `national.overview.topo.json` (all of Canada in EPSG:3347, one object per representation order, `ro1867` … `ro2023`) `canada.topo.json` (2023 outlines, territories included) and `provinces.periods.topo.json` (the outlines by period, as in `data/geo/`). Feature ids are `F_<distid>`.

## Sources

**Provincial results** come from the Provincial Elections Dataset (PED), which reconciles printed and official returns from several sources district by district and grades each contest's confidence. Every source is described on the site's Sources and About pages, with citation and publisher, and listed in `data/sources.parquet`.

**Federal results and boundaries** come from:

Taylor, Zack, Jack Lucas, J.P. Kirby, and Christopher Macdonald Hewitt. 2023. "Canada's Federal Electoral Districts, 1867–2021: New Digital Boundary Files and a Comparative Investigation of District Compactness." *Canadian Journal of Political Science* 56 (2): 451–467. https://doi.org/10.1017/S0008423923000185. [Updated].

## Citing

Taylor, Zack. *Atlas of Canadian Elections: federal and provincial elections in Canada since 1867.* Build 20261106T193703Z. Western University.

When using the federal data, please also cite Taylor, Lucas, Kirby and Macdonald Hewitt (2023), above.
