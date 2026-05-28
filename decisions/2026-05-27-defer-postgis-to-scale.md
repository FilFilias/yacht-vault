---
title: Defer PostGIS — use lat/lng floats + haversine SQL at MVP
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - database
---

# Defer PostGIS — use lat/lng floats + haversine SQL at MVP

## Context

The original architecture spec used a PostGIS `geometry(Point, 4326)` column on the `yachts` table, backed by a GIST index, for geographic search. This requires the `postgis` Postgres extension, the `previewFeatures = ["postgresqlExtensions"]` Prisma flag, and an `Unsupported()` column type that bypasses Prisma's type system.

At MVP, YachtBay launches Greece-only. The total listing count at launch will be in the hundreds, not millions. The query pattern is simple: find active listings within a radius of a lat/lng point.

## Options Considered

- **Option A — PostGIS**: `Unsupported("geometry(Point, 4326)")` column + GIST index. Accurate distance queries in a single SQL statement. Requires Postgres extension + Prisma workaround. Adds operational complexity.
- **Option B — Float columns + haversine SQL**: `locationLat Float` + `locationLng Float` + `$queryRaw` haversine formula + B-tree composite index on `(status, location_lat, location_lng)`. Standard SQL, works out of the box on any Postgres. Less precise at extreme distances (irrelevant in Greece). Scales to ~100k rows without issue.

## Decision

**Option B — Float columns + haversine SQL.**

At Greece-only MVP scale, haversine is accurate enough and operationally simpler. The Prisma schema drops the `postgresqlExtensions` preview feature and the `Unsupported()` type.

## Consequences

- Geo search is slightly less accurate than spherical PostGIS geometry at long distances (not relevant within Greece)
- No Prisma preview features needed
- `$queryRaw` haversine formula used in the Search module for radius filtering
- B-tree composite index on `(status, location_lat, location_lng)` serves both the status filter and the geo bounding-box pre-filter

## Migration path

When listings exceed ~100k or the platform expands beyond Greece:

1. `ALTER TABLE yachts ADD COLUMN location geometry(Point, 4326)`
2. Populate from `location_lat` / `location_lng` with `ST_SetSRID(ST_MakePoint(lng, lat), 4326)`
3. Create GIST index: `CREATE INDEX ON yachts USING GIST (location)`
4. Swap haversine `$queryRaw` for PostGIS `ST_DWithin()` in the Search module
5. Drop float columns (or keep as cache — they become redundant)
6. Enable PostGIS extension + Prisma `postgresqlExtensions` preview feature

This is self-contained in the Search module — no other modules change.

## Related

- [[specs/backend-architecture]]
- [[specs/data-model]]
- [[roadmap/phase-1-development]]
