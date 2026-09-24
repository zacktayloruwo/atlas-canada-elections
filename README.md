# Atlas of Canadian Elections

Provincial general elections in Canada since 1867: every contest and every candidate, mapped on the district boundaries of the day, with district lineages across redistributions.

**Site:** https://zacktayloruwo.github.io/atlas-canada-elections/

This repository holds the published site: a static web app (Svelte, DuckDB-WASM) and the data it reads. It is written by the publish step of the Provincial Elections Dataset (PED) pipeline; nothing is edited here by hand. Build `20260924T204553Z`, pipeline commit `6a9c2c2`, built 2026-09-24T20:45:53Z.

## Using the data

Everything the site shows is in `data/` as Parquet files, readable with DuckDB, Arrow, pandas, R (`arrow`, `nanoparquet`) or any Parquet tool, directly from this repository or from the site URL:

```sql
-- DuckDB
SELECT prabbr, eyear, count(*) AS contests, sum(magnitude) AS seats
FROM 'https://zacktayloruwo.github.io/atlas-canada-elections/data/districts.parquet'
GROUP BY 1, 2 ORDER BY 1, 2;
```

Identifiers: `pedid` is the contest key (province + election year + a sequence, plus a ballot letter where a district returned members on separate ballots). It is not a longitudinal key; `lineage_id` (from `lineages.parquet`) follows a district's territory across boundary changes. Candidates carry no person identifier: rows are grouped by a normalized name within a province (`person_key`, regenerated on every build), and `link_basis` says how each row was attached.

Vote measures are never mixed: `votes_first` and `share_first` are first counts; `votes_final` and `share_final` exist only for alternative-vote and single-transferable-vote contests, and shares only where every candidate has a final count. `quality` grades each contest A to D. `data/manifest.json` carries the definitions of every derived measure.

### Tables

| File | Rows | Columns |
|---|---|---|
| `provinces.parquet` | 10 | prabbr, name_en, name_fr, proj4, snap_tol_m, first_eyear, last_eyear, n_elections |
| `elections.parquet` | 381 | prabbr, eyear, election_date, structure, ballots, formula, n_dist_expected, n_seats_expected, authority, n_dist_returned, n_contests, n_seats_returned, ro_year, has_geometry |
| `districts.parquet` | 21,250 | pedid, prabbr, eyear, pedname_en, pedname_fr, district_base, ballot, geo_role, geo_id, lineage_id, n_contests_on_polygon, magnitude, n_candidates, formula, contested, acclaimed, unfilled, votes_total, votes_total_final, electorate, turnout_pct, margin_first, margin_final, margin_measure, winner_family, seat_split, quality, n_sources |
| `candidates.parquet` | 75,154 | pedid, prabbr, eyear, cluster_id, name, name_key, person_key, link_basis, party_raw, party_source, family, family2, votes_first, votes_final, votes_final_source, share_first, share_final, rank_first, rank_final, elected, quality, slug |
| `seats.parquet` | 22,863 | pedid, prabbr, eyear, cluster_id, seat_index, family, family2 |
| `parties.parquet` | 295 | family, label_en, label_fr, colour_light, colour_dark, pattern, sort_order, n_rows, n_seats, tier |
| `party_labels.parquet` | 913 | prabbr, party_raw, family, family2, first_eyear, last_eyear, n, sources |
| `vacancies.parquet` | 1 | prabbr, eyear, district_name, reason, filled_by, citation |
| `sources.parquet` | 44 | src_abbr, title, publisher, years, provinces, type, url, status, rank, note, citation_en, citation_fr |
| `search_index.parquet` | 104,698 | type, prabbr, display_en, display_fr, key_norm, route |
| `geo_index.parquet` | 9,273 | geo_id, prabbr, ro_year, pednum, pedname, geo_role, bbox_xmin, bbox_ymin, bbox_xmax, bbox_ymax, centroid_x, centroid_y, ov_centroid_x, ov_centroid_y, areakm_net |
| `geo_join.parquet` | 19,354 | pedid, prabbr, eyear, geo_id, geo_role, join_basis |
| `geo_orphans.parquet` | 1,902 | side, prabbr, eyear, ro_year, name, id, reason |
| `era_crosswalk.parquet` | 14,579 | prabbr, ro_from, ro_to, geo_id_from, geo_id_to, area_km2, frac_of_from, frac_of_to, same_lineage |
| `geo_neighbours.parquet` | 21,147 | prabbr, ro_year, geo_id_a, geo_id_b |
| `lineages.parquet` | 2,090 | lineage_id, prabbr, first_eyear, last_eyear, n_contests, n_elections, n_branches_max, names_en, names_fr, lineage_basis, has_floterial, n_eras, n_polygons |
| `source_use.parquet` | 2,072 | prabbr, eyear, field, src_abbr, dq, n |

`data/geo/` holds one TopoJSON per province at full resolution in that province's conic projection (`<PR>.topo.json`, one object per boundary era, arcs shared across eras), an overview version in EPSG:3347 (`<PR>.overview.topo.json`), and `canada.topo.json` with the province outlines. Feature ids are `geo_id` values that `geo_join.parquet` links to contests.

## Sources

Every source is described on the site's About page, with citation and publisher, and listed in `data/sources.parquet`.

## Citing

Taylor, Zack. *Atlas of Canadian Elections: provincial elections in Canada since 1867.* Build 20260924T204553Z. Western University.
