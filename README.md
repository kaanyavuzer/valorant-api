# VALORANT API reference

**Checked:** 2026-11-08  
**Game metadata reported by [valorant-api.com](https://valorant-api.com/v1/version):** `13.09.00.5804228`  
**Client header value reported by the same source:** `release-13.09-shipping-18-5804228`  
**Reported game build date:** `2026-10-28T06:11:26Z`  
**Community OpenAPI document:** [version 0.10.1](https://valdocs.prometheuz.me/openapi.json); 118 HTTP operations over 108 paths in the copy checked on this date.  
**Community spec snapshot:** SHA-256 `70f49e93a23fd05731c97ee02c71663f85f4a40484445407415e3666b09659f4`. We hashed `JSON.stringify()` of the parsed OpenAPI document, not the raw download.

Here's what the available docs say as of this date. It's **not a list of every VALORANT endpoint ever**. Riot's developer API, the unsupported client services, and third-party game data are separate things. The game version alone doesn't tell us whether the client endpoints below still work. We didn't send authenticated requests to Riot or test these flows in the game, and fields, availability, and access rules can change.

### Quick map

| Surface | What it is | Where to look |
|---|---|---|
| Riot Developer API | Supported developer endpoints, subject to Riot's product and player-consent rules | [Official API](#1-riots-official-developer-api) and [official response schemas](#official-response-schema-appendix) |
| Client-facing services | Community-documented `pd`, `shared`, `glz`, SGP and local client routes; unsupported | [Client endpoints](#client-facing-endpoints-unofficial) and [schema appendix](#community-schema-appendix) |
| Riot identity services | Internal, unsupported authentication and session endpoints | [Identity services](#riot-identity-and-session-services-unofficial) |
| `valorant-api.com` | Independent, public static game-data provider; no account data | [Static game data](#2-third-party-static-data-api--valorant-apicom) |

The player examples are made up. We use **`serefsiz#TR1`** as an example, not as a claim that the account exists or owns anything. The static-data snippets can include real public content IDs and shortened values. A PUUID identifies an account; it isn't a username. Riot's developer API uses an **encrypted PUUID**, while community client schemas often show a UUID-shaped `Subject`/`puuid`. Don't treat those formats as interchangeable. A Riot ID also needs a tagline, so swap `TR1` for the real one when you need to.

### Header cheat sheet

**Official developer API:** most calls that use an API key take `X-Riot-Token: <API_KEY>`. `GET /riot/account/v1/accounts/me` takes `Authorization: Bearer <RSO_ACCESS_TOKEN>` instead. An API key, an RSO token, a client access token, and an entitlement token are different things. Having an official API key or RSO access doesn't give you access to the unsupported client endpoints. See [Riot's portal](https://developer.riotgames.com/docs/portal) and [VALORANT policy](https://developer.riotgames.com/docs/valorant).

**Community-documented PD/shared/GLZ calls:** many use the headers below, but not all do. Check each row in the endpoint list for exceptions:

```http
Authorization: Bearer <ACCESS_TOKEN>
X-Riot-Entitlements-JWT: <ENTITLEMENT_TOKEN>
X-Riot-ClientPlatform: <BASE64_ENCODED_PLATFORM_JSON>
X-Riot-ClientVersion: release-13.09-shipping-18-5804228
Content-Type: application/json
```

The platform header is Base64-encoded JSON, not the JSON text itself. Before encoding, an example looks like `{"platformType":"PC","platformOS":"Windows","platformOSVersion":"10.0.26200.1.256.64bit","platformChipset":"Unknown"}`. You need `Content-Type` when sending a JSON body; you don't automatically need it for every GET. [Community platform description](https://valdocs.prometheuz.me/endpoint/fetch-content).

**Local client:** community docs describe HTTPS on `127.0.0.1` with HTTP Basic authentication. The documented header is `Authorization: Basic <base64("riot:" + lockfile_password)>`; the port comes from the Riot Client lockfile too. Base64 is just encoding, not protection. Don't expose the local API, share the lockfile, publish the header, or log account data. [Local help reference](https://valdocs.prometheuz.me/endpoint/local-help).

**Host placeholders:** client docs use `pd.{shard}.a.pvp.net`, `shared.{shard}.a.pvp.net`, `glz-{region}-1.{shard}.a.pvp.net`, and `{cluster}.pp.sgp.pvp.net`. `{region}` and `{shard}` don't always match: the community docs map the `latam` and `br` regions to the `na` shard. Official `*.api.riotgames.com` routing is separate. See [community routing](https://valdocs.prometheuz.me/endpoint/player-session) and Riot's [account reference](https://developer.riotgames.com/api-details/account-v1).

### Community routing: regions, shards, and clusters

The client docs list these region IDs: `na` (North America), `latam` (Latin America), `br` (Brazil), `eu` (Europe), `ap` (Asia Pacific), and `kr` (Korea). They map regions to shards like this:

| Shard | Region(s) in the community docs |
|---|---|
| `na` | `na`, `latam`, `br` |
| `pbe` | `na` (PBE) |
| `eu` | `eu` |
| `ap` | `ap` |
| `kr` | `kr` |

For SGP replay-info requests, the same docs list these clusters:

| Cluster | Region(s) |
|---|---|
| `euc1` | Europe |
| `usw2` | North America, Brazil, Latin America, PBE |
| `apse1` | Asia Pacific, Oceania |
| `apne1` | Japan, Korea |

These are community-documented mappings, not a guarantee for every account or endpoint. Official Riot developer API routes use different routing values. Sources: [player session](https://valdocs.prometheuz.me/endpoint/player-session), [replay info](https://valdocs.prometheuz.me/endpoint/match-replay-info).

### Reading schemas and examples

- Field names and capitalization matter. In the schema appendix, `?` means the source calls a field optional, `null` isn't the same as a missing field, and `unknown` means the source didn't give a shape.
- The TypeScript-like definitions describe fields; they aren't JSON you can send. Unless we say otherwise, the JSON examples are **short examples**, not recordings or complete drop-in responses.
- If a response schema isn't documented, that doesn't mean the response is `{}`, `null`, or empty.
- The community schema appendix follows the source's assumptions. Riot doesn't guarantee them, so check the linked source for numeric limits, formats, descriptions, and updates.
- A source's `required` list doesn't guarantee every field will show up. Riot says its official API may leave out empty strings, zeroes, empty lists, and nulls. Handle missing fields rather than relying on the list alone. [Riot response rules](https://developer.riotgames.com/docs/portal#web-apis_response-codes).
- Riot says non-200 responses may have no body or a non-JSON body. For HTTP `429`, follow `Retry-After` and don't depend on the error text. [Official response and rate-limit notes](https://developer.riotgames.com/docs/portal#web-apis_response-codes).
- Only use accounts and data you're allowed to access. Riot's VALORANT policy requires players to opt in to sharing their data, limits approved uses, and says personal-key applications aren't supported. Check the current [VALORANT policy](https://developer.riotgames.com/docs/valorant).

---

## Riot developer API, static data, and sources

**Reference date:** 2026-11-08
**Public client metadata at that date:** `13.09.00.5804228`, branch `release-13.09`, `release-13.09-shipping-18-5804228` (from `valorant-api.com/v1/version`).
That version comes from a third-party API and describes the game client build. It doesn't prove which undocumented client endpoints exist or still work. The sections below cover what their sources document.

Here's where this section gets its information:
- **[RIOT-OFFICIAL]** — Riot's own Developer Portal (`developer.riotgames.com`).
- **[RIOT-DERIVED]** — `mingweisamuel.com/riotapi-schema`, an OpenAPI document generated daily from Riot's published reference. Not Riot-authored; treated as a mirror of Riot's spec.
- **[THIRD-PARTY-UNOFFICIAL]** — `valorant-api.com`, a community-run static-data host. Not endorsed by Riot.
- **[INDEPENDENT-CROSSCHECK]** — `wardbox.github.io/whisper`, a community SDK's type docs, used only to flag disagreements.

There are no real credentials or account details in these examples. `serefsiz` is a placeholder handle.

---

## 1. Riot's official developer API

### 1.1 Base URLs and routing

VALORANT uses two different routing styles:

- **Regional routes** for `account-v1`: `americas`, `asia`, `europe`. Host form `https://{region}.api.riotgames.com`. **[RIOT-DERIVED / routesTable]**
- **VALORANT platform routes** for every `val-*` endpoint: `ap`, `br`, `eu`, `kr`, `latam`, `na`, and `esports` (a special esports platform). Host form `https://{platform}.api.riotgames.com`. **[RIOT-DERIVED]**

These VALORANT platform routes aren't the same as League of Legends platform IDs such as `na1` or `tr1`, so don't mix them up.

### 1.2 Authentication

- Standard API-key access: send the key in the `X-Riot-Token` **header**, or as the `api_key` **query parameter** (query form is for browser debugging only). **[RIOT-DERIVED securitySchemes]**
- RSO (Riot Sign On) access token access: the `/riot/account/v1/accounts/me` endpoint is called with `Authorization: Bearer {accessToken}` instead of an API key. **[RIOT-OFFICIAL — curl examples in the VALORANT docs page]**
- RSO is only available to Production-level API keys, and Riot states all VALORANT apps must have players opt in to data sharing via RSO. **[RIOT-OFFICIAL]**

### 1.3 Endpoint table — account-v1 (regional routing)

| Method | Path | Description | Path/query params |
| --- | --- | --- | --- |
| GET | `/riot/account/v1/accounts/by-riot-id/{gameName}/{tagLine}` | Look up an account (and its PUUID) by Riot ID | `gameName`, `tagLine` (path, required) |
| GET | `/riot/account/v1/accounts/by-puuid/{puuid}` | Look up a Riot ID by PUUID | `puuid` (path, required) |
| GET | `/riot/account/v1/accounts/me` | Account for the RSO access token | RSO Bearer token; no API key |
| GET | `/riot/account/v1/active-shards/by-game/{game}/by-puuid/{puuid}` | Active shard for a player | `game` = `val`\|`lor`\|`2xko`; `puuid` |
| GET | `/riot/account/v1/region/by-game/{game}/by-puuid/{puuid}` | Active region (LoL/TFT only) | `game` = `lol`\|`tft`; `puuid` |

**[RIOT-DERIVED]** — `x-platforms-available` for all five: `americas`, `asia`, `europe`. `by-riot-id` is marked `x-nullable-404`.

Riot's portal also shows esports-labelled versions of the first three account calls at the same paths. The mirror only models the regional versions above, so this reference can't establish the esports-specific routing or access rules.

### 1.4 Endpoint table — val-match-v1 (VALORANT platform routing)

| Method | Path | Description | Params |
| --- | --- | --- | --- |
| GET | `/val/match/v1/matches/{matchId}` | Match detail | `matchId` (path) |
| GET | `/val/match/v1/matchlists/by-puuid/{puuid}` | Match history for a player | `puuid` (path) |
| GET | `/val/match/v1/recent-matches/by-queue/{queue}` | Recent match IDs for a queue | `queue` (path, enum) |

`x-platforms-available`: `ap`, `br`, `esports`, `eu`, `kr`, `latam`, `na`.
`recent-matches` `queue` enum: `competitive`, `unrated`, `spikerush`, `tournamentmode`, `deathmatch`, `onefa`, `ggteam`, `hurm`, `swiftplay`, `skirmish2v2`, `skirmishascension1v1`, `skirmishascension2v2`, `test`, `abilitydraftarena`.
Implementation note from the spec: recent-matches returns IDs completed in the last 10 minutes for live regions and 12 hours for the esports route; NA/LATAM/BR share a match-history deployment and return a combined list; requests are load-balanced and may be inconsistent. **[RIOT-DERIVED]**

### 1.5 Endpoint table — val-content-v1, val-ranked-v1, val-status-v1

| Method | Path | Description | Params |
| --- | --- | --- | --- |
| GET | `/val/content/v1/contents` | Game content, optionally localized | `locale` (query, optional) |
| GET | `/val/ranked/v1/leaderboards/by-act/{actId}` | Competitive-queue leaderboard | `actId` (path); `size` (1–200, default 200); `startIndex` (default 0) |
| GET | `/val/status/v1/platform-data` | Platform status, incidents, maintenances | none |

- `val-content-v1` platforms: `ap`, `br`, `esports`, `eu`, `kr`, `latam`, `na`.
- `val-ranked-v1` platforms: `ap`, `br`, `eu`, `kr`, `latam`, `na`. `x-nullable-404`. `actId` note: "Act ids can be found using the val-content API."
- `val-status-v1`: operation `getPlatformData` at `/val/status/v1/platform-data`, confirmed on Riot's `val-status-v1` reference page. **[RIOT-OFFICIAL]**

### 1.6 Endpoint table — console APIs (VALORANT platform routing)

| Method | Path | `platformType` required? | Params |
| --- | --- | --- | --- |
| GET | `/val/match/console/v1/matches/{matchId}` | **No** | `matchId` (path) |
| GET | `/val/match/console/v1/matchlists/by-puuid/{puuid}` | **Yes** | `puuid` (path), `platformType` (query, required) |
| GET | `/val/match/console/v1/recent-matches/by-queue/{queue}` | **No** | `queue` (path, enum) |
| GET | `/val/console/ranked/v1/leaderboards/by-act/{actId}` | **Yes** | `actId` (path), `platformType` (query, required), `startIndex`, `size` |

`platformType` enum: `playstation`, `xbox`. **[RIOT-DERIVED + RIOT-OFFICIAL console page]**

**Heads up: the sources disagree.** The whisper SDK docs say every console endpoint needs `platformType`. **[INDEPENDENT-CROSSCHECK]** Riot's own pages only list it for **matchlist** and the **leaderboard**; they don't list it for `matches/{matchId}` or `recent-matches/by-queue/{queue}`. The table follows Riot's pages, but keep the disagreement in mind.
Console-match platforms: `ap`, `br`, `eu`, `latam`, `na`. Console-ranked platforms: `ap`, `eu`, `na`.
Console `recent-matches` `queue` enum uses `console_`-prefixed values (`console_unrated`, `console_swiftplay`, `console_hurm`, `console_deathmatch`, `console_competitive`, `console_skirmish2v2`, `console_skirmishascension1v1`, `console_skirmishascension2v2`, `console_abilitydraftarena`).

### 1.7 val-accolades-v1

Riot's Developer Portal lists a `val-accolades-v1` resource with two operations: **[RIOT-OFFICIAL — from the portal's `val-accolades-v1` reference page]**

| Method | Path | Description | Params |
| --- | --- | --- | --- |
| GET | `/val/accolades/v1/accolades` | Get accolades | not documented on the page |
| GET | `/val/accolades/v1/players/{puuid}` | Get player accolades | `puuid` (path, required) |

Riot's page shows **no response class** for either call, only the standard errors, so we don't have documented response bodies. It lists `puuid` as required for `players/{puuid}`; the `accolades` page doesn't give parameter details.

The Riot-derived OpenAPI mirror and the whisper type reference don't list these accolades paths. The two calls above come from Riot's own portal.

### 1.8 Response DTOs (from the Riot-derived spec)

**account-v1.AccountDto** (required: `puuid`)
```
puuid    string   "Encrypted PUUID. Exact length of 78 characters."
gameName string   may be omitted if the account has no gameName
tagLine  string   may be omitted if the account has no tagLine
```

**val-content-v1.ContentDto** (required: `version`, `characters`, `maps`, `chromas`, `skins`, `skinLevels`, `equips`, `gameModes`, `sprays`, `sprayLevels`, `charms`, `charmLevels`, `playerCards`, `playerTitles`, `acts`). All listed arrays except `acts` contain `ContentItemDto` objects; `acts` contains `ActDto` objects. `ContentItemDto` fields include `name`, `localizedNames` (excluded when a locale is set), `id`, `assetName`, and `assetPath` (maps and game modes only).

**val-match-v1.MatchDto** (required: `matchInfo`, `players`, `coaches`): also `teams`, `roundResults`.
`MatchInfoDto` required: `matchId`, `mapId`, `gameVersion`, `region`, `gameStartMillis`, `provisioningFlowId`, `isCompleted`, `customGameName`, `queueId`, `gameMode`, `isRanked`, `seasonId`, `premierMatchInfo`; plus `gameLengthMillis`.
`PlayerDto` required: `puuid`, `gameName`, `tagLine`, `teamId`, `partyId`, `competitiveTier`, `isObserver`, `playerCard`, `playerTitle`, `accountLevel`; plus `characterId`, `stats`.

**val-match-v1.MatchlistDto** (required: `puuid`, `history`) — `history` is a list of `MatchlistEntryDto` (`matchId`, `gameStartTimeMillis`, `queueId`).

**val-match-v1.RecentMatchesDto** (required: `currentTime`, `matchIds`).

**val-ranked-v1.LeaderboardDto** (required: `shard`, `actId`, `totalPlayers`, `players`): also `immortalStartingPage`, `immortalStartingIndex`, `topTierRRThreshold`, `tierDetails` (map of tier → `TierDetailDto`), `startIndex`, `query`.
`PlayerDto` required: `leaderboardRank`, `rankedRating`, `numberOfWins`, `premierRosterType`; `puuid`/`gameName`/`tagLine` "may be omitted if the player has been anonymized."

**val-status-v1.PlatformDataDto** (required: `id`, `name`, `locales`, `maintenances`, `incidents`). `StatusDto`: `id`, `maintenance_status` (`scheduled`|`in_progress`|`complete`), `incident_severity` (`info`|`warning`|`critical`), `titles`, `updates`, `created_at`, `archive_at`, `updated_at`, `platforms`.

### 1.9 Standard error statuses

All operations in the mirror list these errors: `400`, `401`, `403`, `404`, `405`, `415`, `429`, `500`, `502`, `503`, `504`. A `429` means you're rate-limited; follow the `Retry-After` header. **[RIOT-DERIVED + RIOT-OFFICIAL getting-started page]**

### 1.10 Illustrative request/response (partial, fictional field values)

The JSON below follows the `AccountDto` shape, but we made up the values. It's **not** a real account.

```
GET https://europe.api.riotgames.com/riot/account/v1/accounts/by-riot-id/serefsiz/TR1
Header: X-Riot-Token: <your-development-key>
```
```
{
  "puuid": "000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gameName": "serefsiz",
  "tagLine": "TR1"
}
```
The `puuid` above is 78 zeroes to match the documented length. A real PUUID is an opaque encrypted string.

---

## 2. Third-party static data API — valorant-api.com

**[THIRD-PARTY-UNOFFICIAL]** Base URL: `https://valorant-api.com/v1`. You don't need an API key. This community-run service gives you game content and assets such as agents, weapons, and maps; it doesn't give you player data or match history and isn't supported by Riot.

These endpoints returned `200` with a `{"status":200,"data":...}` envelope when checked:

| Path | What it returns |
| --- | --- |
| `/v1/version` | Client branch/version metadata (values shown in the header above) |
| `/v1/agents` | Agents; supports `isPlayableCharacter=true` and `language=` |
| `/v1/weapons` | Weapons with `weaponStats` and `shopData` |
| `/v1/maps` | Maps with coordinates and `callouts` |
| `/v1/sprays` | Sprays and spray levels |
| `/v1/bundles` | Store bundles |
| `/v1/playercards` | Player cards |
| `/v1/buddies` | Gun buddies ("charms") |

**No top-level skins endpoint was confirmed:** `/v1/skins` returned HTTP 404 when checked. Skins show up inside each weapon's `skins` array under `/v1/weapons`.

The pages checked don't give a numeric rate limit or promise that IDs and fields will stay the same, so don't rely on either assumption.

### Illustrative shapes (partial, truncated — field values are real API output, list contents abbreviated)

`/v1/agents` item (abridged; actual responses carry many more fields and full ability text):
```
{
  "status": 200,
  "data": [
    {
      "uuid": "e370fa57-4757-3604-3648-499e1f642d3f",
      "displayName": "Gekko",
      "description": "...",
      "isPlayableCharacter": true,
      "abilities": [
        { "slot": "Ability1", "displayName": "Wingman", "description": "..." }
      ]
    }
  ]
}
```

`/v1/weapons` item (abridged):
```
{
  "uuid": "63e6c2b6-4a8e-869c-3d4c-e38355226584",
  "displayName": "Odin",
  "category": "EEquippableCategory::Heavy",
  "weaponStats": { "fireRate": 12, "magazineSize": 100, "damageRanges": [ ... ] },
  "shopData": { "cost": 3200, "category": "Heavy Weapons" },
  "skins": [ ... ]
}
```

`/v1/maps` item (abridged):
```
{
  "uuid": "7eaecc1b-4337-bbf6-6ab9-04b8f06b3319",
  "displayName": "Ascent",
  "assetPath": "ShooterGame/Content/Maps/Ascent/Ascent_PrimaryAsset",
  "callouts": [ { "regionName": "Tree", "superRegionName": "A" } ]
}
```

`language` is a query parameter (e.g. `?language=tr-TR`) that localizes `displayName`/`description` fields.

---

## 3. PUUID vs Riot ID

- A **Riot ID** is the player-facing handle, the pair `gameName` + `tagLine` (e.g. `serefsiz#TR1`). Requests pass them as two separate path segments: `/accounts/by-riot-id/{gameName}/{tagLine}`.
- A **PUUID** is the opaque account identifier returned in `AccountDto.puuid`, documented as an encrypted string of exactly 78 characters. Match, matchlist, and leaderboard lookups key on PUUID.
- Convert Riot ID → PUUID with `account-v1` `by-riot-id`; convert PUUID → Riot ID with `account-v1` `by-puuid`. Riot's own guidance to third parties is to display Riot IDs and store PUUID as the join key, and to prefer PUUID-based endpoints. **[RIOT-OFFICIAL]**
- If an account has no `gameName`/`tagLine`, the corresponding field may be absent from the response. **[RIOT-DERIVED]**

---

## 4. Limitations and provenance

1. **Version:** `13.09.00.5804228` comes from the third-party `/v1/version` endpoint. It's a client build number, not proof that Riot's official or undocumented endpoints work with it.
2. **Riot-derived mirror:** `mingweisamuel.com/riotapi-schema` is generated from Riot's reference, but Riot doesn't host or endorse it. Where it differs from Riot's portal, as with accolades, both are noted above.
3. **Unofficial docs:** `valdocs.prometheuz.me` isn't an official source. Its `0.10.1` version is the docs-package version, not a tested game build. `wardbox.github.io/whisper` is a community SDK reference used here only to point out the console `platformType` disagreement.
4. **Missing responses:** Riot's portal lists the `val-accolades-v1` calls without response schemas. Their response bodies remain unknown here.
5. **Static API:** `valorant-api.com` only provides content/assets, not a player's match history, rank, or account identity. It doesn't need an API key.
6. **No bypass:** this reference doesn't describe or require bypassing authentication, anti-cheat, or access control.

---

## 5. Sources

Primary (Riot Developer Portal):
- API index and VALORANT docs: https://developer.riotgames.com/apis , https://developer.riotgames.com/docs/valorant
- Per-endpoint reference pages (RIOT-OFFICIAL, primary):
  - account-v1: https://developer.riotgames.com/api-details/account-v1
  - val-match-v1: https://developer.riotgames.com/api-details/val-match-v1
  - val-content-v1: https://developer.riotgames.com/api-details/val-content-v1
  - val-ranked-v1: https://developer.riotgames.com/api-details/val-ranked-v1
  - val-status-v1: https://developer.riotgames.com/api-details/val-status-v1
  - val-console-match-v1: https://developer.riotgames.com/api-details/val-console-match-v1
  - val-console-ranked-v1: https://developer.riotgames.com/api-details/val-console-ranked-v1
  - val-accolades-v1: https://developer.riotgames.com/api-details/val-accolades-v1
- Getting-started (API key types, response codes, rate limiting): https://developer.riotgames.com/getting-started.html

Supporting:
- Riot-generated OpenAPI mirror (RIOT-DERIVED): https://www.mingweisamuel.com/riotapi-schema/openapi-3.0.0.min.json
- Riot-derived routing table (RIOT-DERIVED): https://www.mingweisamuel.com/riotapi-schema/routesTable.json
- Third-party static data API (THIRD-PARTY-UNOFFICIAL): https://valorant-api.com/v1
- Independent cross-check (INDEPENDENT-CROSSCHECK): https://wardbox.github.io/whisper/docs/api/val
- Unofficial doc mirror, cited as unofficial only: https://valdocs.prometheuz.me/openapi.json

---

## Worked examples (fictional and redacted)

These examples show the request and response shapes, not a live recording. `serefsiz#TR1` and the IDs are placeholders. Only replace them with data you're allowed to use. The client schemas can change or disappear, so an example doesn't prove it still works.

### Riot developer API: account by Riot ID

```http
GET https://europe.api.riotgames.com/riot/account/v1/accounts/by-riot-id/serefsiz/TR1
X-Riot-Token: <API_KEY>
```

Illustrative response shaped like the documented `AccountDto`:

```json
{
  "puuid": "000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "gameName": "serefsiz",
  "tagLine": "TR1"
}
```

The example encrypted PUUID is 78 zeroes, not a real identifier. A real value is opaque. See [Riot's account reference](https://developer.riotgames.com/api-details/account-v1).

### Client name service: look up a UUID

```http
PUT https://pd.eu.a.pvp.net/name-service/v2/players
Authorization: Bearer <ACCESS_TOKEN>
X-Riot-Entitlements-JWT: <ENTITLEMENT_TOKEN>
X-Riot-ClientPlatform: <BASE64_ENCODED_PLATFORM_JSON>
X-Riot-ClientVersion: release-13.09-shipping-18-5804228
Content-Type: application/json
```

```json
["00000000-0000-0000-0000-000000000000"]
```

The community OpenAPI says the response is an array of name records. The webpage shows the item fields, while its machine-readable schema shows an **array** at the top level:

```json
[
  {
    "DisplayName": "serefsiz",
    "Subject": "00000000-0000-0000-0000-000000000000",
    "GameName": "serefsiz",
    "TagLine": "TR1"
  }
]
```

See [Name Service](https://valdocs.prometheuz.me/endpoint/name-service), request [S016](#schema-s016), and response [S017](#schema-s017). This UUID-shaped `Subject` is not the example encrypted PUUID from the official API.

### Client store: storefront v3

```http
POST https://pd.eu.a.pvp.net/store/v3/storefront/00000000-0000-0000-0000-000000000000
Authorization: Bearer <ACCESS_TOKEN>
X-Riot-Entitlements-JWT: <ENTITLEMENT_TOKEN>
X-Riot-ClientPlatform: <BASE64_ENCODED_PLATFORM_JSON>
X-Riot-ClientVersion: release-13.09-shipping-18-5804228
Content-Type: application/json
```

The community schema says this body is required and lists it as an object with no properties:

```json
{}
```

The documented response is the bigger [S033](#schema-s033) object, including `FeaturedBundle`, `SkinsPanelLayout`, and other fields. A short excerpt isn't a complete mock response. The older community source lists `GET /store/v2/storefront/{puuid}` instead; we didn't try either route with a signed-in client. [v3 source](https://valdocs.prometheuz.me/endpoint/storefront) · [v2 source](https://valapidocs.techchrism.me/endpoint/storefront).

### Local client: account alias

```http
GET https://127.0.0.1:{port}/player-account/aliases/v1/active
Authorization: Basic <BASE64_OF_riot_COLON_PASSWORD>
```

Illustrative response, following the community [S077](#schema-s077) shape:

```json
{
  "active": true,
  "created_datetime": 0,
  "game_name": "serefsiz",
  "summoner": false,
  "tag_line": "TR1"
}
```

The local password and Basic header are credentials. Don't paste, publish, or log them. The client binds to loopback, so don't expose its port. [Local API source](https://valdocs.prometheuz.me/endpoint/account-alias).

---

## Client-facing endpoints (unofficial)

This list comes from the community [OpenAPI 0.10.1](https://valdocs.prometheuz.me/openapi.json), checked on 2026-11-08. It tells you what the docs say, not what we measured in a running client. The schema links show documented JSON bodies or HTTP 200 responses; we label documented HTTP 204 responses separately. `—` means the source doesn't give a JSON schema, not that the request or response is necessarily empty. Check the OpenAPI link for parameter types and exact limits.

| Base | Host |
|---|---|
| PD | `https://pd.{shard}.a.pvp.net` |
| SHARED | `https://shared.{shard}.a.pvp.net` |
| GLZ | `https://glz-{region}-1.{shard}.a.pvp.net` |
| SGP | `https://{cluster}.pp.sgp.pvp.net` |
| LOCAL | `https://127.0.0.1:{port}` |

**Headers:** `C` = Bearer access token + `X-Riot-Entitlements-JWT` + `X-Riot-ClientPlatform` + `X-Riot-ClientVersion`. `E` = Bearer + `X-Riot-Entitlements-JWT`. `B` = Bearer only. `L` = local Basic. `N` = none listed in the source. JSON bodies use `Content-Type: application/json`; the table tells you when a body is required and links its schema. These codes reflect the docs, not a guarantee about today's game client.

In the query column, `!` means the source says the parameter is required; `?` means it's optional. Placeholders such as `{puuid}` and `{matchId}` belong in the path. The OpenAPI source says which ones are required.

The source says Account XP only accepts the signed-in player's PUUID. The replay-info page says the player must be you or a friend. The list doesn't establish other access rules, so don't assume you can call a route just because it's listed.

The community leaderboard URL below hard-codes `/affinity/na/`. That's the source's exact example; it doesn't prove every shard uses `na` or that other affinity values work.

### Player data, history and configuration

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Fetch Content](https://valdocs.prometheuz.me/endpoint/fetch-content) | `GET SHARED /content-service/v3/content` | C | — | — | [S001](#schema-s001) |
| [Account XP](https://valdocs.prometheuz.me/endpoint/account-xp) | `GET PD /account-xp/v1/players/{puuid}` | C | — | — | [S002](#schema-s002) |
| [Player Loadout](https://valdocs.prometheuz.me/endpoint/player-loadout) | `GET PD /personalization/v3/players/{puuid}/playerloadout` | C | — | — | [S003](#schema-s003) |
| [Set Player Loadout](https://valdocs.prometheuz.me/endpoint/set-player-loadout) | `PUT PD /personalization/v3/players/{puuid}/playerloadout` | C | — | [S004](#schema-s004) (required) | [S003](#schema-s003) |
| [Player MMR](https://valdocs.prometheuz.me/endpoint/player-mmr) | `GET PD /mmr/v1/players/{puuid}` | C | — | — | [S005](#schema-s005) |
| [Match History](https://valdocs.prometheuz.me/endpoint/match-history) | `GET PD /match-history/v1/history/{puuid}` | C | `startIndex?`, `endIndex?`, `queue?` | — | [S006](#schema-s006) |
| [Match Details](https://valdocs.prometheuz.me/endpoint/match-details) | `GET PD /match-details/v1/matches/{matchID}` | C | — | — | [S007](#schema-s007) |
| [Competitive Updates](https://valdocs.prometheuz.me/endpoint/competitive-updates) | `GET PD /mmr/v1/players/{puuid}/competitiveupdates` | C | `startIndex?`, `endIndex?`, `queue?` | — | [S008](#schema-s008) |
| [Player Region Index](https://valdocs.prometheuz.me/endpoint/player-region-index) | `GET PD /region-index/v1/players/{puuid}` | C | `seasonID!` | — | [S009](#schema-s009) |
| [Leaderboard](https://valdocs.prometheuz.me/endpoint/leaderboard) | `GET PD /mmr/v1/leaderboards/affinity/na/queue/competitive/season/{seasonId}` | C | `startIndex!`, `size!`, `query?` | — | [S010](#schema-s010) |
| [Penalties](https://valdocs.prometheuz.me/endpoint/penalties) | `GET PD /restrictions/v3/penalties` | C | — | — | [S011](#schema-s011) |
| [Player Interventions](https://valdocs.prometheuz.me/endpoint/player-interventions) | `GET PD /restrictions/v1/activeFutureInterventions` | C | — | — | [S012](#schema-s012) |
| [Player Report Token](https://valdocs.prometheuz.me/endpoint/player-report-token) | `GET PD /restrictions/v1/playerReportToken/{matchid}/{offenderpuuid}` | C | — | — | [S013](#schema-s013) |
| [Player Avoid List](https://valdocs.prometheuz.me/endpoint/player-avoid-list) | `GET PD /restrictions/v1/avoidList` | C | — | — | [S014](#schema-s014) |
| [Config](https://valdocs.prometheuz.me/endpoint/config) | `GET SHARED /v1/config/{region}` | N | — | — | [S015](#schema-s015) |
| [Name Service](https://valdocs.prometheuz.me/endpoint/name-service) | `PUT PD /name-service/v2/players` | C | — | [S016](#schema-s016) (required) | [S017](#schema-s017) |
| [Item Upgrades](https://valdocs.prometheuz.me/endpoint/item-upgrades) | `GET PD /contract-definitions/v3/item-upgrades` | C | — | — | [S070](#schema-s070) |
| [Contracts](https://valdocs.prometheuz.me/endpoint/contracts) | `GET PD /contracts/v1/contracts/{puuid}` | C | — | — | [S071](#schema-s071) |
| [Activate Contract](https://valdocs.prometheuz.me/endpoint/activate-contract) | `POST PD /contracts/v1/contracts/{puuid}/special/{contractId}` | C | — | — | [S071](#schema-s071) |
| [Contract Match](https://valdocs.prometheuz.me/endpoint/contract-match) | `GET PD /contracts/v1/players/{puuid}/matches/{matchId}` | C | — | — | [S072](#schema-s072) |

### Store and entitlements

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Storefront](https://valdocs.prometheuz.me/endpoint/storefront) | `POST PD /store/v3/storefront/{puuid}` | C | — | [S032](#schema-s032) (required) | [S033](#schema-s033) |
| [Create Order](https://valdocs.prometheuz.me/endpoint/create-order) | `POST PD /store/v1/order/` | C | — | [S034](#schema-s034) (required) | [S035](#schema-s035) |
| [Get Order](https://valdocs.prometheuz.me/endpoint/get-order) | `GET PD /store/v1/order/{orderId}` | C | — | — | [S036](#schema-s036) |
| [Wallet](https://valdocs.prometheuz.me/endpoint/wallet) | `GET PD /store/v1/wallet/{puuid}` | C | — | — | [S037](#schema-s037) |
| [Owned Items](https://valdocs.prometheuz.me/endpoint/owned-items) | `GET PD /store/v1/entitlements/{puuid}/{ItemTypeID}` | C | — | — | [S038](#schema-s038) |
| [All Owned Items](https://valdocs.prometheuz.me/endpoint/all-owned-items) | `GET PD /store/v1/entitlements/{puuid}` | C | — | — | [S038](#schema-s038) |
| [Gift Purchaser Eligibility](https://valdocs.prometheuz.me/endpoint/gift-purchaser-eligibility) | `GET PD /store/v1/gifts/{puuid}/purchasereligibility` | C | — | — | [S039](#schema-s039) |
| [Gift Recipient Eligibility](https://valdocs.prometheuz.me/endpoint/gift-recipient-eligibility) | `POST PD /store/v1/gifts/{puuid}/eligibility/{recipientpuuid}` | C | — | — | [S040](#schema-s040) |
| [Agent Storefront](https://valdocs.prometheuz.me/endpoint/agent-storefront) | `GET PD /store/v1/storefronts/agent` | C | — | — | [S041](#schema-s041) |

### Party and session

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Player Session](https://valdocs.prometheuz.me/endpoint/player-session) | `GET GLZ /session/v1/sessions/{puuid}` | C | — | — | [S018](#schema-s018) |
| [Party](https://valdocs.prometheuz.me/endpoint/party) | `GET GLZ /parties/v1/parties/{partyId}` | C | — | — | [S019](#schema-s019) |
| [Party Player](https://valdocs.prometheuz.me/endpoint/party-player) | `GET GLZ /parties/v1/players/{puuid}` | C | — | — | [S020](#schema-s020) |
| [Party Remove Player](https://valdocs.prometheuz.me/endpoint/party-remove-player) | `DELETE GLZ /parties/v1/players/{puuid}` | C | — | — | HTTP 204; no body |
| [Party Set Member Ready](https://valdocs.prometheuz.me/endpoint/party-set-member-ready) | `POST GLZ /parties/v1/parties/{partyId}/members/{puuid}/setReady` | C | — | [S021](#schema-s021) (required) | [S019](#schema-s019) |
| [Refresh Competitive Tier](https://valdocs.prometheuz.me/endpoint/refresh-competitive-tier) | `POST GLZ /parties/v1/parties/{partyId}/members/{puuid}/refreshCompetitiveTier` | C | — | — | [S019](#schema-s019) |
| [Refresh Player Identity](https://valdocs.prometheuz.me/endpoint/refresh-player-identity) | `POST GLZ /parties/v1/parties/{partyId}/members/{puuid}/refreshPlayerIdentity` | C | — | — | [S019](#schema-s019) |
| [Refresh Pings](https://valdocs.prometheuz.me/endpoint/refresh-pings) | `POST GLZ /parties/v1/parties/{partyId}/members/{puuid}/refreshPings` | C | — | — | [S019](#schema-s019) |
| [Change Queue](https://valdocs.prometheuz.me/endpoint/change-queue) | `POST GLZ /parties/v1/parties/{partyId}/queue` | C | — | [S022](#schema-s022) (required) | [S019](#schema-s019) |
| [Start Custom Game](https://valdocs.prometheuz.me/endpoint/start-custom-game) | `POST GLZ /parties/v1/parties/{partyId}/startcustomgame` | C | — | — | [S019](#schema-s019) |
| [Start Solo Experience](https://valdocs.prometheuz.me/endpoint/start-solo-experience) | `POST GLZ /parties/v1/players/{puuid}/startsoloexperience` | C | — | [S023](#schema-s023) (required) | [S020](#schema-s020) |
| [Enter Matchmaking Queue](https://valdocs.prometheuz.me/endpoint/enter-matchmaking-queue) | `POST GLZ /parties/v1/parties/{partyId}/matchmaking/join` | C | — | — | [S019](#schema-s019) |
| [Leave Matchmaking Queue](https://valdocs.prometheuz.me/endpoint/leave-matchmaking-queue) | `POST GLZ /parties/v1/parties/{partyId}/matchmaking/leave` | C | — | — | [S019](#schema-s019) |
| [Set Party Accessibility](https://valdocs.prometheuz.me/endpoint/set-party-accessibility) | `POST GLZ /parties/v1/parties/{partyId}/accessibility` | C | — | [S024](#schema-s024) (required) | [S019](#schema-s019) |
| [Set Custom Game Settings](https://valdocs.prometheuz.me/endpoint/set-custom-game-settings) | `POST GLZ /parties/v1/parties/{partyId}/customgamesettings` | C | — | [S025](#schema-s025) (required) | [S019](#schema-s019) |
| [Custom Game Membership](https://valdocs.prometheuz.me/endpoint/custom-game-membership) | `POST GLZ /parties/v1/parties/{partyId}/customgamemembership/{team}` | C | — | [S026](#schema-s026) (required) | [S019](#schema-s019) |
| [Set Player Moderator Status](https://valdocs.prometheuz.me/endpoint/set-player-moderator-status) | `POST GLZ /parties/v1/parties/{partyId}/setplayermoderatorstatus` | C | — | [S027](#schema-s027) (required) | [S019](#schema-s019) |
| [Set Player Broadcast HUD Status](https://valdocs.prometheuz.me/endpoint/set-player-broadcast-hud-status) | `POST GLZ /parties/v1/parties/{partyId}/setplayerbroadcasthudstatus` | C | — | [S028](#schema-s028) (required) | [S019](#schema-s019) |
| [Party Invite](https://valdocs.prometheuz.me/endpoint/party-invite) | `POST GLZ /parties/v1/parties/{partyId}/invites/name/{name}/tag/{tagline}` | C | — | — | [S019](#schema-s019) |
| [Party Request](https://valdocs.prometheuz.me/endpoint/party-request) | `POST GLZ /parties/v1/parties/{partyId}/request` | C | — | — | [S029](#schema-s029) |
| [Party Decline](https://valdocs.prometheuz.me/endpoint/party-decline) | `POST GLZ /parties/v1/parties/{partyId}/request/{requestId}/decline` | C | — | — | [S019](#schema-s019) |
| [Custom Game Configs](https://valdocs.prometheuz.me/endpoint/custom-game-configs) | `GET GLZ /parties/v1/parties/customgameconfigs` | C | — | — | [S030](#schema-s030) |
| [Party Chat Token](https://valdocs.prometheuz.me/endpoint/party-chat-token) | `GET GLZ //parties/v1/parties/{partyId}/muctoken` | C | — | — | [S031](#schema-s031) |
| [Party Voice Token](https://valdocs.prometheuz.me/endpoint/party-voice-token) | `GET GLZ //parties/v1/parties/{partyId}/voicetoken` | C | — | — | [S031](#schema-s031) |
| [Party Disable Code](https://valdocs.prometheuz.me/endpoint/party-disable-code) | `DELETE GLZ /parties/v1/parties/{partyId}/invitecode` | C | — | — | [S019](#schema-s019) |
| [Party Generate Code](https://valdocs.prometheuz.me/endpoint/party-generate-code) | `POST GLZ /parties/v1/parties/{partyId}/invitecode` | C | — | — | [S019](#schema-s019) |
| [Party Join By Code](https://valdocs.prometheuz.me/endpoint/party-join-by-code) | `POST GLZ /parties/v1/players/joinbycode/{code}` | C | — | — | [S020](#schema-s020) |

### Agent select and live game

The community source warns against using the select/lock calls for automatic instalocking; it says this may lead to bans or API restrictions. Their presence here is not an endorsement.

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Pre-Game Player](https://valdocs.prometheuz.me/endpoint/pre-game-player) | `GET GLZ /pregame/v1/players/{puuid}` | C | — | — | [S042](#schema-s042) |
| [Pre-Game Match](https://valdocs.prometheuz.me/endpoint/pre-game-match) | `GET GLZ /pregame/v1/matches/{preGameMatchId}` | C | — | — | [S043](#schema-s043) |
| [Pre-Game Loadouts](https://valdocs.prometheuz.me/endpoint/pre-game-loadouts) | `GET GLZ /pregame/v1/matches/{preGameMatchId}/loadouts` | C | — | — | [S044](#schema-s044) |
| [Select Character](https://valdocs.prometheuz.me/endpoint/select-character) | `POST GLZ /pregame/v1/matches/{preGameMatchId}/select/{agentId}` | C | — | — | [S043](#schema-s043) |
| [Lock Character](https://valdocs.prometheuz.me/endpoint/lock-character) | `POST GLZ /pregame/v1/matches/{preGameMatchId}/lock/{agentId}` | C | — | — | [S043](#schema-s043) |
| [Pre-Game Quit](https://valdocs.prometheuz.me/endpoint/pre-game-quit) | `POST GLZ /pregame/v1/matches/{preGameMatchId}/quit` | C | — | — | HTTP 204; no body |
| [Current Game Player](https://valdocs.prometheuz.me/endpoint/current-game-player) | `GET GLZ /core-game/v1/players/{puuid}` | C | — | — | [S042](#schema-s042) |
| [Current Game Match](https://valdocs.prometheuz.me/endpoint/current-game-match) | `GET GLZ /core-game/v1/matches/{currentGameMatchId}` | C | — | — | [S045](#schema-s045) |
| [Current Game Loadouts](https://valdocs.prometheuz.me/endpoint/current-game-loadouts) | `GET GLZ /core-game/v1/matches/{currentGameMatchId}/loadouts` | C | — | — | [S046](#schema-s046) |
| [Current Game Quit](https://valdocs.prometheuz.me/endpoint/current-game-quit) | `POST GLZ /core-game/v1/players/{puuid}/disassociate/{currentGameMatchId}` | C | — | — | HTTP 204; no body |

### Premier, esports and progression

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Premier Player](https://valdocs.prometheuz.me/endpoint/premier-player) | `GET PD /premier/v2/players/{puuid}` | C | — | — | [S047](#schema-s047) |
| [Premier Player Crests](https://valdocs.prometheuz.me/endpoint/premier-player-crests) | `GET PD /premier/v2/players/{puuid}/crests` | C | — | — | [S048](#schema-s048) |
| [Premier Eligibility](https://valdocs.prometheuz.me/endpoint/premier-eligibility) | `GET PD /premier/v1/player/eligibility` | C | — | — | [S049](#schema-s049) |
| [Premier Seasons](https://valdocs.prometheuz.me/endpoint/premier-seasons) | `GET PD /premier/v1/affinities/{affinity}/premier-seasons` | C | — | — | [S050](#schema-s050) |
| [Premier Season Active](https://valdocs.prometheuz.me/endpoint/premier-season-active) | `GET PD /premier/v1/affinities/{affinity}/premier-seasons/active` | C | — | — | [S051](#schema-s051) |
| [Premier Conferences](https://valdocs.prometheuz.me/endpoint/premier-conferences) | `GET PD /premier/v1/affinities/{affinity}/conferences` | C | — | — | [S052](#schema-s052) |
| [Upcoming Matches](https://valdocs.prometheuz.me/endpoint/upcoming-matches) | `GET PD /esports-service/v2/upcomingMatches` | E | `leagueID!`, `locale?`, `sport?` | — | [S053](#schema-s053) |
| [Matches](https://valdocs.prometheuz.me/endpoint/matches) | `POST PD /esports-service/v2/matches` | E | `locale?`, `sport?` | [S054](#schema-s054) (required) | [S055](#schema-s055) |
| [Roster](https://valdocs.prometheuz.me/endpoint/roster) | `GET PD /premier/v2/rosters/{rosterid}` | C | — | — | [S056](#schema-s056) |
| [Premier Roster Match History](https://valdocs.prometheuz.me/endpoint/premier-roster-match-history) | `GET PD /premier/v1/rosters/{rosterid}/matchhistory` | C | — | — | [S057](#schema-s057) |
| [Premier Roster Public Info](https://valdocs.prometheuz.me/endpoint/premier-roster-public-info) | `GET PD /premier/v2/rosters/{rosterid}/publicInfo` | C | — | — | [S058](#schema-s058) |
| [Premier Roster Customization](https://valdocs.prometheuz.me/endpoint/premier-roster-customization) | `PUT PD /premier/v1/rosters/{rosterid}/customization` | C | — | [S059](#schema-s059) (required) | [S029](#schema-s029) |
| [Goldstars](https://valdocs.prometheuz.me/endpoint/goldstars) | `GET PD /goldstars/v1/goldstars` | C | — | — | [S062](#schema-s062) |
| [Player Goldstars](https://valdocs.prometheuz.me/endpoint/player-goldstars) | `GET PD /goldstars/v1/players/{puuid}` | C | — | — | [S063](#schema-s063) |
| [Bloomline Player](https://valdocs.prometheuz.me/endpoint/bloomline-player) | `GET PD /bloomline/v0/players/{puuid}` | C | — | — | [S067](#schema-s067) |
| [Bloomline Player Bonsai](https://valdocs.prometheuz.me/endpoint/bloomline-player-bonsai) | `GET PD /bloomline/v0/players/{puuid}/bonsai/{agentId}` | C | — | — | [S067](#schema-s067) |
| [Update Bloomline Player Bonsai](https://valdocs.prometheuz.me/endpoint/update-bloomline-player-bonsai) | `POST PD /bloomline/v0/players/{puuid}/bonsai/{agentId}` | C | — | [S068](#schema-s068) (required) | [S067](#schema-s067) |
| [Bloomline Valid Queues](https://valdocs.prometheuz.me/endpoint/bloomline-valid-queues) | `GET PD /bloomline/v0/config/valid-queues` | C | — | — | [S069](#schema-s069) |

### Mailbox and activities

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Mailbox](https://valdocs.prometheuz.me/endpoint/mailbox) | `GET SGP /mailbox/v1/{puuid}/product/valorant` | B | `count?`, `startIndex?`, `includedStates?` | — | [S060](#schema-s060) |
| [Mail](https://valdocs.prometheuz.me/endpoint/mail) | `GET SGP /mailbox/v1/{puuid}/mail/{mailId}` | B | — | — | [S061](#schema-s061) |
| [Delete Mail](https://valdocs.prometheuz.me/endpoint/delete-mail) | `DELETE SGP /mailbox/v1/{puuid}/mail/{mailId}` | B | — | — | HTTP 204; no body |
| [Activity Lookup](https://valdocs.prometheuz.me/endpoint/activity-lookup) | `POST SGP /activities/v1/lookup` | B | — | [S064](#schema-s064) (required) | [S065](#schema-s065) |
| [Shared Join Link](https://valdocs.prometheuz.me/endpoint/shared-join-link) | `POST SGP /activities/v1/{activityId}/join-codes/shared` | B | — | — | [S066](#schema-s066) |
| [Disable Join Link](https://valdocs.prometheuz.me/endpoint/disable-join-link) | `DELETE SGP /activities/v1/{activityId}/join-codes/shared` | B | — | — | HTTP 204; no body |

### Local Riot Client and chat

| Operation | Method and path | Headers | Query | Request JSON | HTTP 200 JSON |
|---|---|---|---|---|---|
| [Local Help](https://valdocs.prometheuz.me/endpoint/local-help) | `GET LOCAL /help` | L | — | — | [S073](#schema-s073) |
| [Sessions](https://valdocs.prometheuz.me/endpoint/sessions) | `GET LOCAL /product-session/v1/external-sessions` | L | — | — | [S074](#schema-s074) |
| [RSO User Info](https://valdocs.prometheuz.me/endpoint/rso-user-info) | `GET LOCAL /rso-auth/v1/authorization/userinfo` | L | — | — | [S075](#schema-s075) |
| [Client Region](https://valdocs.prometheuz.me/endpoint/client-region) | `GET LOCAL /riotclient/region-locale` | L | — | — | [S076](#schema-s076) |
| [Account Alias](https://valdocs.prometheuz.me/endpoint/account-alias) | `GET LOCAL /player-account/aliases/v1/active` | L | — | — | [S077](#schema-s077) |
| [Entitlements Token](https://valdocs.prometheuz.me/endpoint/entitlements-token) | `GET LOCAL /entitlements/v1/token` | L | — | — | [S078](#schema-s078) |
| [Chat Session](https://valdocs.prometheuz.me/endpoint/chat-session) | `GET LOCAL /chat/v1/session` | L | — | — | [S079](#schema-s079) |
| [Friends](https://valdocs.prometheuz.me/endpoint/friends) | `GET LOCAL /chat/v4/friends` | L | — | — | [S080](#schema-s080) |
| [Send Friend Request](https://valdocs.prometheuz.me/endpoint/send-friend-request) | `POST LOCAL /chat/v4/friendrequests` | L | — | [S081](#schema-s081) (required) | [S082](#schema-s082) |
| [Remove Friend Request](https://valdocs.prometheuz.me/endpoint/remove-friend-request) | `DELETE LOCAL /chat/v4/friendrequests` | L | — | [S083](#schema-s083) (required) | HTTP 204; no body |
| [Friend Requests](https://valdocs.prometheuz.me/endpoint/friend-requests) | `GET LOCAL /chat/v4/friendrequests` | L | — | — | [S084](#schema-s084) |
| [Presence](https://valdocs.prometheuz.me/endpoint/presence) | `GET LOCAL /chat/v4/presences` | L | — | — | [S085](#schema-s085) |
| [Local Swagger Docs](https://valdocs.prometheuz.me/endpoint/local-swagger-docs) | `GET LOCAL /swagger/v3/openapi.json` | L | — | — | [S086](#schema-s086) |
| [Party Chat Info](https://valdocs.prometheuz.me/endpoint/party-chat-info) | `GET LOCAL /chat/v6/conversations/ares-parties` | L | — | — | [S087](#schema-s087) |
| [Pre-Game Chat Info](https://valdocs.prometheuz.me/endpoint/pre-game-chat-info) | `GET LOCAL /chat/v6/conversations/ares-pregame` | L | — | — | [S087](#schema-s087) |
| [Current Game Chat Info](https://valdocs.prometheuz.me/endpoint/current-game-chat-info) | `GET LOCAL /chat/v6/conversations/ares-coregame` | L | — | — | [S087](#schema-s087) |
| [All Chat Info](https://valdocs.prometheuz.me/endpoint/all-chat-info) | `GET LOCAL /chat/v6/conversations` | L | — | — | [S087](#schema-s087) |
| [Chat Participants](https://valdocs.prometheuz.me/endpoint/chat-participants) | `GET LOCAL /chat/v5/participants` | L | `cid?` | — | [S088](#schema-s088) |
| [Send Chat](https://valdocs.prometheuz.me/endpoint/send-chat) | `POST LOCAL /chat/v6/messages` | L | — | [S089](#schema-s089) (required) | [S090](#schema-s090) |
| [Chat History](https://valdocs.prometheuz.me/endpoint/chat-history) | `GET LOCAL /chat/v6/messages` | L | `cid?` | — | [S090](#schema-s090) |

The community spec writes the party chat/voice token paths as `//parties/...`. An [older endpoint list](https://raw.githubusercontent.com/liamcottle/valorant.js/master/endpoints.txt) uses one slash for party chat. We haven't checked which spelling a particular build accepts.

### Extra documented routes and version drift

| Route | What the source describes | Source |
|---|---|---|
| `GET PD /store/v2/storefront/{puuid}` | Older storefront; the newer community catalog instead lists `POST /store/v3/storefront/{puuid}` with a required JSON object. Treat these as separate, unverified versions. | [Older storefront](https://valapidocs.techchrism.me/endpoint/storefront) · [Newer storefront](https://valdocs.prometheuz.me/endpoint/storefront) |
| `GET` / `PUT PD /personalization/v2/players/{puuid}/playerloadout` | Older loadout schema uses `Sprays`; community v3 uses `ActiveExpressions` and `DynamicOptions`. | [v2 read](https://valapidocs.techchrism.me/endpoint/player-loadout) · [v2 write](https://valapidocs.techchrism.me/endpoint/set-player-loadout) · [v3 read](https://valdocs.prometheuz.me/endpoint/player-loadout) |
| `GET PD /store/v1/offers/` | Older source describes an `Offers` array with `OfferID`, `IsDirectPurchase`, `StartDate`, `Cost`, and `Rewards`. | [Prices](https://valapidocs.techchrism.me/endpoint/prices) |
| `GET SGP /match-history-query/v3/products/valorant/players/{puuid}/infoTypes/{type}` | Query repeats `id`; documented response includes `total` and `matchFileUrlsMap`. The signed URLs are sensitive. This page is not in the checked OpenAPI inventory. | [Replay info](https://valdocs.prometheuz.me/endpoint/match-replay-info) |

The [older community endpoint list](https://raw.githubusercontent.com/liamcottle/valorant.js/master/endpoints.txt) also mentions session connect/heartbeat, pregame and core-game chat/voice tokens, contract definitions, party variants, login queue, and latency endpoints. It doesn't provide full request/response schemas there, so don't assume they're still current.

The newer community OpenAPI lists five calls that don't appear in the [2024-04-20 Insomnia export](https://valapidocs.techchrism.me/insomnia.json): `startsoloexperience`, `customgamemembership/{team}`, `setplayermoderatorstatus`, `setplayerbroadcasthudstatus`, and `GET /session/v1/sessions/{puuid}`. They're in the newer catalog above, but we haven't checked how they behave in any client build. Not appearing in the old export doesn't prove they didn't exist.

**What's still unknown:** the checked sources don't give full HTTP 200 JSON shapes for [S029](#schema-s029) or [S086](#schema-s086); the `val-accolades-v1` response bodies are also undocumented on Riot's portal. The community sources don't provide a complete XMPP stanza schema, local WebSocket event schema, or a reliable general rate-limit and error-body contract. The six HTTP 204 responses shown in the catalog are explicitly documented as no-content; that's different from an unknown JSON shape.

---

## Riot identity and session services (unofficial)

These are client services described by the community, **not** Riot's supported developer API. For a real third-party product, use Riot's approved [RSO integration](https://developer.riotgames.com/docs/valorant#rso-integration) and get the player's opt-in. Don't ask anyone to hand over a password, MFA code, session cookie, bearer token, or entitlement token. Keep credentials out of logs, Markdown, and chat. Responses and access rules can change without notice.

| Method and endpoint | Request / headers | Documented response or result | Source |
|---|---|---|---|
| `POST https://auth.riotgames.com/api/v1/authorization` | `Content-Type: application/json`; fields `client_id`, `nonce`, `redirect_uri`, `response_type`, `scope` | Sets up the authorization session; JSON response shape is not documented in this source. | [Auth Cookies](https://valdocs.prometheuz.me/endpoint/auth-cookies) |
| `PUT https://auth.riotgames.com/api/v1/authorization` | `Content-Type: application/json`; auth-state object | The community schema describes `type: success` with `success` / `country` / `platform`, or `type: multifactor` with `multifactor` / `country` / `platform` and an optional `error`. Treat `login_token`, redirect URLs and user identity fields as sensitive. | [Auth Request](https://valapidocs.techchrism.me/endpoint/auth-request) |
| `PUT https://auth.riotgames.com/api/v1/authorization` | MFA-state object, same URL; keep the code private | An invalid MFA code may still produce HTTP 200 and an `error` field; this is a community observation, not a guaranteed contract. The final response schema is not fully documented. | [MFA](https://valapidocs.techchrism.me/endpoint/multi-factor-authentication) |
| `GET https://auth.riotgames.com/authorize` | Existing authenticated session; `redirect_uri`, `client_id`, `response_type`, `nonce`, `scope` query parameters | Community docs describe a redirect: success can include sensitive token material in its URL fragment; failure can redirect to login. Do not log the `Location` header. | [Cookie Reauth](https://valapidocs.techchrism.me/endpoint/cookie-reauth) |
| `POST https://entitlements.auth.riotgames.com/api/token/v1` | `Authorization: Bearer <ACCESS_TOKEN>`; community page also lists `Content-Type: application/json` | `{ "entitlements_token": "<REDACTED>" }` | [Entitlement](https://valapidocs.techchrism.me/endpoint/entitlement) |
| `GET https://auth.riotgames.com/userinfo` | `Authorization: Bearer <ACCESS_TOKEN>` | Object includes `sub` (PUUID), `acct.game_name`, `acct.tag_line`, `country`, `player_locale`, `affinity`, and other sensitive account metadata. | [Player Info](https://valapidocs.techchrism.me/endpoint/player-info) |
| `PUT https://riot-geo.pas.si.riotgames.com/pas/v1/product/valorant` | `Authorization: Bearer <ACCESS_TOKEN>`; JSON object with `id_token` | Object with sensitive `token` and `affinities: { pbe, live }`. | [Riot Geo](https://valapidocs.techchrism.me/endpoint/riot-geo) |
| `GET https://riot-geo.pas.si.riotgames.com/pas/v1/service/chat` | `Authorization: Bearer <ACCESS_TOKEN>` | PAS token string. It is a credential; keep it private. | [PAS Token](https://valapidocs.techchrism.me/endpoint/pas-token) |
| `GET https://clientconfig.rpg.riotgames.com/api/v1/config/player?app=Riot%20Client` | `Authorization: Bearer <ACCESS_TOKEN>` and `X-Riot-Entitlements-JWT: <ENTITLEMENTS_TOKEN>` | Object with `chat.affinities`, `chat.affinity_domains`, `chat.port`, and other configuration properties. | [Riot Client Config](https://valapidocs.techchrism.me/endpoint/riot-client-config) |

For the `PUT /api/v1/authorization` request, the community schema includes `type`, `language`, `remember`, and a `riot_identity` object with `captcha`, `username`, and `password`. In a fictional example the username would be `"serefsiz"`; don't put a real password, CAPTCHA token, MFA code, or session token in this document. This is an unsupported client-auth schema, not a recommended way to sign users in. Use Riot's approved RSO flow for third-party apps. [Community schema](https://valapidocs.techchrism.me/endpoint/auth-request).

Community docs say chat uses a TLS-protected TCP connection carrying XML, not a JSON REST endpoint. XML can arrive in pieces, and tokens and messages are sensitive. See [XMPP Connection](https://valapidocs.techchrism.me/endpoint/xmpp-connection). We didn't test authentication or chat against the game build listed above.

### Local events and chat transport

The older community [Insomnia export](https://valapidocs.techchrism.me/insomnia.json) includes a local WebSocket template at `wss://127.0.0.1:{port}`, using the same Basic credentials, with subscription payload `[5,"OnJsonApiEvent"]`. It doesn't give a complete event schema, and we haven't checked event names, payloads, or availability against this game build.

The `chat.affinities`, `chat.affinity_domains`, and `chat.port` config fields describe the chat host and port. The older XMPP page says port `5223` was observed, not guaranteed. It doesn't give a complete stanza schema. The local plugin WebSocket URLs in the export are proxy tools, not Riot API endpoints.

#### XMPP proxy templates in the older export

These are **local plugin templates**, not Riot server URLs. The export uses the placeholder names shown below; the direct-connection templates list a bearer token, `X-Riot-Entitlements-JWT`, and `X-Riot-PAS-JWT`, while the MITM templates list no headers. Don't copy or log token values. Source: [2024 Insomnia export](https://valapidocs.techchrism.me/insomnia.json).

| Template | Local plugin URL | What it does |
|---|---|---|
| XMPP Raw Connection | `{% xmpp_websocket_url %}/raw` | Raw XML; messages can arrive in pieces |
| XMPP Raw Buffered Connection | `{% xmpp_websocket_url %}/raw-buffered` | Buffers until a full XML message arrives |
| XMPP JSON Connection | `{% xmpp_websocket_url %}/json` | Buffers and converts XML to JSON |
| XMPP Raw MITM | `{% xmpp_mitm_websocket_url %}/raw` | Raw XML through the plugin's MITM proxy |
| XMPP Raw Buffered MITM | `{% xmpp_mitm_websocket_url %}/raw-buffered` | Buffered XML through the proxy |
| XMPP JSON MITM | `{% xmpp_mitm_websocket_url %}/json` | XML converted to JSON through the proxy |

The placeholder URLs are provided by [insomnia-plugin-valorant](https://github.com/techchrism/insomnia-plugin-valorant). They aren't separate Riot endpoints, and none was tested here.

---

## Official response schema appendix

These TypeScript-like types come from a [community mirror of Riot's API reference](https://www.mingweisamuel.com/riotapi-schema/openapi-3.0.0.min.json), checked on 2026-11-08. They show the documented field names and types, not a live response. `?` follows the mirror's optional fields; `unknown` means it didn't give a type. We use `O_` and underscores to turn the source names into TypeScript identifiers. We leave out descriptions, formats, ranges, and most other OpenAPI constraints, so check Riot's linked pages for those details.

The mirror doesn't include the two accolades calls, and Riot's portal doesn't show their response classes. We don't know their response bodies from these sources.

### Route to response type

| Method and path | Response type |
|---|---|
| `GET /riot/account/v1/accounts/by-puuid/{puuid}` | `O_account_v1_AccountDto` |
| `GET /riot/account/v1/accounts/by-riot-id/{gameName}/{tagLine}` | `O_account_v1_AccountDto` |
| `GET /riot/account/v1/accounts/me` | `O_account_v1_AccountDto` |
| `GET /riot/account/v1/active-shards/by-game/{game}/by-puuid/{puuid}` | `O_account_v1_ActiveShardDto` |
| `GET /riot/account/v1/region/by-game/{game}/by-puuid/{puuid}` | `O_account_v1_AccountRegionDTO` |
| `GET /val/match/console/v1/matches/{matchId}` | `O_val_console_match_v1_MatchDto` |
| `GET /val/match/console/v1/matchlists/by-puuid/{puuid}` | `O_val_console_match_v1_MatchlistDto` |
| `GET /val/match/console/v1/recent-matches/by-queue/{queue}` | `O_val_console_match_v1_RecentMatchesDto` |
| `GET /val/console/ranked/v1/leaderboards/by-act/{actId}` | `O_val_console_ranked_v1_LeaderboardDto` |
| `GET /val/content/v1/contents` | `O_val_content_v1_ContentDto` |
| `GET /val/match/v1/matches/{matchId}` | `O_val_match_v1_MatchDto` |
| `GET /val/match/v1/matchlists/by-puuid/{puuid}` | `O_val_match_v1_MatchlistDto` |
| `GET /val/match/v1/recent-matches/by-queue/{queue}` | `O_val_match_v1_RecentMatchesDto` |
| `GET /val/ranked/v1/leaderboards/by-act/{actId}` | `O_val_ranked_v1_LeaderboardDto` |
| `GET /val/status/v1/platform-data` | `O_val_status_v1_PlatformDataDto` |


### Type definitions

```ts
// account-v1.AccountDto
type O_account_v1_AccountDto = {
  "puuid": string;
  "gameName"?: string;
  "tagLine"?: string;
};

// account-v1.AccountRegionDTO
type O_account_v1_AccountRegionDTO = {
  "puuid": string;
  "game": string;
  "region": string;
};

// account-v1.ActiveShardDto
type O_account_v1_ActiveShardDto = {
  "puuid": string;
  "game": string;
  "activeShard": string;
};

// val-console-match-v1.AbilityCastsDto
type O_val_console_match_v1_AbilityCastsDto = {
  "grenadeCasts": number;
  "ability1Casts": number;
  "ability2Casts": number;
  "ultimateCasts": number;
};

// val-console-match-v1.AbilityDto
type O_val_console_match_v1_AbilityDto = {
  "grenadeEffects"?: string;
  "ability1Effects"?: string;
  "ability2Effects"?: string;
  "ultimateEffects"?: string;
};

// val-console-match-v1.CoachDto
type O_val_console_match_v1_CoachDto = {
  "puuid": string;
  "teamId": string;
};

// val-console-match-v1.DamageDto
type O_val_console_match_v1_DamageDto = {
  "receiver": string;
  "damage": number;
  "legshots": number;
  "bodyshots": number;
  "headshots": number;
};

// val-console-match-v1.EconomyDto
type O_val_console_match_v1_EconomyDto = {
  "loadoutValue": number;
  "weapon": string;
  "armor": string;
  "remaining": number;
  "spent": number;
};

// val-console-match-v1.FinishingDamageDto
type O_val_console_match_v1_FinishingDamageDto = {
  "damageType": string;
  "damageItem": string;
  "isSecondaryFireMode": boolean;
};

// val-console-match-v1.KillDto
type O_val_console_match_v1_KillDto = {
  "timeSinceGameStartMillis": number;
  "timeSinceRoundStartMillis": number;
  "killer": string;
  "victim": string;
  "victimLocation": O_val_console_match_v1_LocationDto;
  "assistants": Array<string>;
  "playerLocations": Array<O_val_console_match_v1_PlayerLocationsDto>;
  "finishingDamage": O_val_console_match_v1_FinishingDamageDto;
};

// val-console-match-v1.LocationDto
type O_val_console_match_v1_LocationDto = {
  "x": number;
  "y": number;
};

// val-console-match-v1.MatchDto
type O_val_console_match_v1_MatchDto = {
  "matchInfo": O_val_console_match_v1_MatchInfoDto;
  "players": Array<O_val_console_match_v1_PlayerDto>;
  "coaches": Array<O_val_console_match_v1_CoachDto>;
  "teams"?: Array<O_val_console_match_v1_TeamDto>;
  "roundResults"?: Array<O_val_console_match_v1_RoundResultDto>;
};

// val-console-match-v1.MatchInfoDto
type O_val_console_match_v1_MatchInfoDto = {
  "matchId": string;
  "mapId": string;
  "gameLengthMillis"?: number;
  "gameStartMillis": number;
  "provisioningFlowId": string;
  "isCompleted": boolean;
  "customGameName": string;
  "queueId": string;
  "gameMode": string;
  "isRanked": boolean;
  "seasonId": string;
};

// val-console-match-v1.MatchlistDto
type O_val_console_match_v1_MatchlistDto = {
  "puuid": string;
  "history": Array<O_val_console_match_v1_MatchlistEntryDto>;
};

// val-console-match-v1.MatchlistEntryDto
type O_val_console_match_v1_MatchlistEntryDto = {
  "matchId": string;
  "gameStartTimeMillis": number;
  "queueId": string;
};

// val-console-match-v1.PlayerDto
type O_val_console_match_v1_PlayerDto = {
  "puuid": string;
  "gameName": string;
  "tagLine": string;
  "teamId": string;
  "partyId": string;
  "characterId"?: string;
  "stats"?: O_val_console_match_v1_PlayerStatsDto;
  "competitiveTier": number;
  "playerCard": string;
  "playerTitle": string;
};

// val-console-match-v1.PlayerLocationsDto
type O_val_console_match_v1_PlayerLocationsDto = {
  "puuid": string;
  "viewRadians": number;
  "location": O_val_console_match_v1_LocationDto;
};

// val-console-match-v1.PlayerRoundStatsDto
type O_val_console_match_v1_PlayerRoundStatsDto = {
  "puuid": string;
  "kills": Array<O_val_console_match_v1_KillDto>;
  "damage": Array<O_val_console_match_v1_DamageDto>;
  "score": number;
  "economy": O_val_console_match_v1_EconomyDto;
  "ability": O_val_console_match_v1_AbilityDto;
};

// val-console-match-v1.PlayerStatsDto
type O_val_console_match_v1_PlayerStatsDto = {
  "score": number;
  "roundsPlayed": number;
  "kills": number;
  "deaths": number;
  "assists": number;
  "playtimeMillis": number;
  "abilityCasts"?: O_val_console_match_v1_AbilityCastsDto;
};

// val-console-match-v1.RecentMatchesDto
type O_val_console_match_v1_RecentMatchesDto = {
  "currentTime": number;
  "matchIds": Array<string>;
};

// val-console-match-v1.RoundResultDto
type O_val_console_match_v1_RoundResultDto = {
  "roundNum": number;
  "roundResult": string;
  "roundCeremony": string;
  "winningTeam": string;
  "bombPlanter"?: string;
  "bombDefuser"?: string;
  "plantRoundTime": number;
  "plantPlayerLocations"?: Array<O_val_console_match_v1_PlayerLocationsDto>;
  "plantLocation": O_val_console_match_v1_LocationDto;
  "plantSite": string;
  "defuseRoundTime": number;
  "defusePlayerLocations"?: Array<O_val_console_match_v1_PlayerLocationsDto>;
  "defuseLocation": O_val_console_match_v1_LocationDto;
  "playerStats": Array<O_val_console_match_v1_PlayerRoundStatsDto>;
  "roundResultCode": string;
};

// val-console-match-v1.TeamDto
type O_val_console_match_v1_TeamDto = {
  "teamId": string;
  "won": boolean;
  "roundsPlayed": number;
  "roundsWon": number;
  "numPoints": number;
};

// val-console-ranked-v1.LeaderboardDto
type O_val_console_ranked_v1_LeaderboardDto = {
  "actId": string;
  "totalPlayers": number;
  "query"?: string;
  "shard": string;
  "players": Array<O_val_console_ranked_v1_PlayerDto>;
  "tierDetails"?: Array<O_val_console_ranked_v1_TierDto>;
};

// val-console-ranked-v1.PlayerDto
type O_val_console_ranked_v1_PlayerDto = {
  "puuid"?: string;
  "gameName"?: string;
  "tagLine"?: string;
  "leaderboardRank": number;
  "rankedRating": number;
  "numberOfWins": number;
};

// val-console-ranked-v1.TierDto
type O_val_console_ranked_v1_TierDto = object;

// val-content-v1.ActDto
type O_val_content_v1_ActDto = {
  "name": string;
  "localizedNames"?: O_val_content_v1_LocalizedNamesDto;
  "id": string;
  "isActive": boolean;
  "parentId"?: string;
  "type"?: string;
};

// val-content-v1.ContentDto
type O_val_content_v1_ContentDto = {
  "version": string;
  "characters": Array<O_val_content_v1_ContentItemDto>;
  "maps": Array<O_val_content_v1_ContentItemDto>;
  "chromas": Array<O_val_content_v1_ContentItemDto>;
  "skins": Array<O_val_content_v1_ContentItemDto>;
  "skinLevels": Array<O_val_content_v1_ContentItemDto>;
  "equips": Array<O_val_content_v1_ContentItemDto>;
  "gameModes": Array<O_val_content_v1_ContentItemDto>;
  "sprays": Array<O_val_content_v1_ContentItemDto>;
  "sprayLevels": Array<O_val_content_v1_ContentItemDto>;
  "charms": Array<O_val_content_v1_ContentItemDto>;
  "charmLevels": Array<O_val_content_v1_ContentItemDto>;
  "playerCards": Array<O_val_content_v1_ContentItemDto>;
  "playerTitles": Array<O_val_content_v1_ContentItemDto>;
  "acts": Array<O_val_content_v1_ActDto>;
  "ceremonies"?: Array<O_val_content_v1_ContentItemDto>;
  "totems"?: Array<O_val_content_v1_ContentItemDto>;
};

// val-content-v1.ContentItemDto
type O_val_content_v1_ContentItemDto = {
  "name": string;
  "localizedNames"?: O_val_content_v1_LocalizedNamesDto;
  "id": string;
  "assetName": string;
  "assetPath"?: string;
};

// val-content-v1.LocalizedNamesDto
type O_val_content_v1_LocalizedNamesDto = {
  "ar-AE": string;
  "de-DE": string;
  "en-GB"?: string;
  "en-US": string;
  "es-ES": string;
  "es-MX": string;
  "fr-FR": string;
  "id-ID": string;
  "it-IT": string;
  "ja-JP": string;
  "ko-KR": string;
  "pl-PL": string;
  "pt-BR": string;
  "ru-RU": string;
  "th-TH": string;
  "tr-TR": string;
  "vi-VN": string;
  "zh-CN": string;
  "zh-TW": string;
};

// val-match-v1.AbilityCastsDto
type O_val_match_v1_AbilityCastsDto = {
  "grenadeCasts": number;
  "ability1Casts": number;
  "ability2Casts": number;
  "ultimateCasts": number;
};

// val-match-v1.AbilityDto
type O_val_match_v1_AbilityDto = {
  "grenadeEffects"?: string;
  "ability1Effects"?: string;
  "ability2Effects"?: string;
  "ultimateEffects"?: string;
};

// val-match-v1.CoachDto
type O_val_match_v1_CoachDto = {
  "puuid": string;
  "teamId": string;
};

// val-match-v1.DamageDto
type O_val_match_v1_DamageDto = {
  "receiver": string;
  "damage": number;
  "legshots": number;
  "bodyshots": number;
  "headshots": number;
};

// val-match-v1.EconomyDto
type O_val_match_v1_EconomyDto = {
  "loadoutValue": number;
  "weapon": string;
  "armor": string;
  "remaining": number;
  "spent": number;
};

// val-match-v1.FinishingDamageDto
type O_val_match_v1_FinishingDamageDto = {
  "damageType": string;
  "damageItem": string;
  "isSecondaryFireMode": boolean;
};

// val-match-v1.KillDto
type O_val_match_v1_KillDto = {
  "timeSinceGameStartMillis": number;
  "timeSinceRoundStartMillis": number;
  "killer": string;
  "victim": string;
  "victimLocation": O_val_match_v1_LocationDto;
  "assistants": Array<string>;
  "playerLocations": Array<O_val_match_v1_PlayerLocationsDto>;
  "finishingDamage": O_val_match_v1_FinishingDamageDto;
};

// val-match-v1.LocationDto
type O_val_match_v1_LocationDto = {
  "x": number;
  "y": number;
};

// val-match-v1.MatchDto
type O_val_match_v1_MatchDto = {
  "matchInfo": O_val_match_v1_MatchInfoDto;
  "players": Array<O_val_match_v1_PlayerDto>;
  "coaches": Array<O_val_match_v1_CoachDto>;
  "teams"?: Array<O_val_match_v1_TeamDto>;
  "roundResults"?: Array<O_val_match_v1_RoundResultDto>;
};

// val-match-v1.MatchInfoDto
type O_val_match_v1_MatchInfoDto = {
  "matchId": string;
  "mapId": string;
  "gameVersion": string;
  "gameLengthMillis"?: number;
  "region": string;
  "gameStartMillis": number;
  "provisioningFlowId": string;
  "isCompleted": boolean;
  "customGameName": string;
  "queueId": string;
  "gameMode": string;
  "isRanked": boolean;
  "seasonId": string;
  "premierMatchInfo": object;
};

// val-match-v1.MatchlistDto
type O_val_match_v1_MatchlistDto = {
  "puuid": string;
  "history": Array<O_val_match_v1_MatchlistEntryDto>;
};

// val-match-v1.MatchlistEntryDto
type O_val_match_v1_MatchlistEntryDto = {
  "matchId": string;
  "gameStartTimeMillis": number;
  "queueId": string;
};

// val-match-v1.PlayerDto
type O_val_match_v1_PlayerDto = {
  "puuid": string;
  "gameName": string;
  "tagLine": string;
  "teamId": string;
  "partyId": string;
  "characterId"?: string;
  "stats"?: O_val_match_v1_PlayerStatsDto;
  "competitiveTier": number;
  "isObserver": boolean;
  "playerCard": string;
  "playerTitle": string;
  "accountLevel": number;
};

// val-match-v1.PlayerLocationsDto
type O_val_match_v1_PlayerLocationsDto = {
  "puuid": string;
  "viewRadians": number;
  "location": O_val_match_v1_LocationDto;
};

// val-match-v1.PlayerRoundStatsDto
type O_val_match_v1_PlayerRoundStatsDto = {
  "puuid": string;
  "kills": Array<O_val_match_v1_KillDto>;
  "damage": Array<O_val_match_v1_DamageDto>;
  "score": number;
  "economy": O_val_match_v1_EconomyDto;
  "ability": O_val_match_v1_AbilityDto;
};

// val-match-v1.PlayerStatsDto
type O_val_match_v1_PlayerStatsDto = {
  "score": number;
  "roundsPlayed": number;
  "kills": number;
  "deaths": number;
  "assists": number;
  "playtimeMillis": number;
  "abilityCasts"?: O_val_match_v1_AbilityCastsDto;
};

// val-match-v1.RecentMatchesDto
type O_val_match_v1_RecentMatchesDto = {
  "currentTime": number;
  "matchIds": Array<string>;
};

// val-match-v1.RoundResultDto
type O_val_match_v1_RoundResultDto = {
  "roundNum": number;
  "roundResult": string;
  "roundCeremony": string;
  "winningTeam": string;
  "winningTeamRole": string;
  "bombPlanter"?: string;
  "bombDefuser"?: string;
  "plantRoundTime": number;
  "plantPlayerLocations"?: Array<O_val_match_v1_PlayerLocationsDto>;
  "plantLocation": O_val_match_v1_LocationDto;
  "plantSite": string;
  "defuseRoundTime": number;
  "defusePlayerLocations"?: Array<O_val_match_v1_PlayerLocationsDto>;
  "defuseLocation": O_val_match_v1_LocationDto;
  "playerStats": Array<O_val_match_v1_PlayerRoundStatsDto>;
  "roundResultCode": string;
};

// val-match-v1.TeamDto
type O_val_match_v1_TeamDto = {
  "teamId": string;
  "won": boolean;
  "roundsPlayed": number;
  "roundsWon": number;
  "numPoints": number;
};

// val-ranked-v1.LeaderboardDto
type O_val_ranked_v1_LeaderboardDto = {
  "shard": string;
  "actId": string;
  "totalPlayers": number;
  "players": Array<O_val_ranked_v1_PlayerDto>;
  "immortalStartingPage"?: number;
  "immortalStartingIndex"?: number;
  "topTierRRThreshold"?: number;
  "tierDetails"?: Record<string, O_val_ranked_v1_TierDetailDto>;
  "startIndex"?: number;
  "query"?: string;
};

// val-ranked-v1.PlayerDto
type O_val_ranked_v1_PlayerDto = {
  "puuid"?: string;
  "gameName"?: string;
  "tagLine"?: string;
  "leaderboardRank": number;
  "rankedRating": number;
  "numberOfWins": number;
  "competitiveTier"?: number;
  "prefix"?: string;
  "premierRosterType": string;
};

// val-ranked-v1.TierDetailDto
type O_val_ranked_v1_TierDetailDto = {
  "rankedRatingThreshold": number;
  "startingPage": number;
  "startingIndex": number;
};

// val-status-v1.ContentDto
type O_val_status_v1_ContentDto = {
  "locale": string;
  "content": string;
};

// val-status-v1.PlatformDataDto
type O_val_status_v1_PlatformDataDto = {
  "id": string;
  "name": string;
  "locales": Array<string>;
  "maintenances": Array<O_val_status_v1_StatusDto>;
  "incidents": Array<O_val_status_v1_StatusDto>;
};

// val-status-v1.StatusDto
type O_val_status_v1_StatusDto = {
  "id": number;
  "maintenance_status": "scheduled" | "in_progress" | "complete";
  "incident_severity": "info" | "warning" | "critical";
  "titles": Array<O_val_status_v1_ContentDto>;
  "updates": Array<O_val_status_v1_UpdateDto>;
  "created_at": string;
  "archive_at": string;
  "updated_at": string;
  "platforms": Array<"windows" | "macos" | "android" | "ios" | "ps4" | "xbone" | "switch">;
};

// val-status-v1.UpdateDto
type O_val_status_v1_UpdateDto = {
  "id": number;
  "author": string;
  "publish": boolean;
  "publish_locations": Array<"riotclient" | "riotstatus" | "game">;
  "translations": Array<O_val_status_v1_ContentDto>;
  "created_at": string;
  "updated_at": string;
};

```

---

## Community schema appendix

These types are a readable version of the JSON schemas in the [community OpenAPI 0.10.1](https://valdocs.prometheuz.me/openapi.json), checked on 2026-11-08. They aren't live responses. `S###` names are labels we use in this file. `?` means the source calls a field optional; a field without it is required in that source. `unknown` means the source didn't give a type. We keep nested fields, `null`, constants, and enums, but leave out details like patterns, ranges, timestamps, `additionalProperties: false`, and descriptions. Check the original schema for those. Real endpoint access, inputs, and status codes can differ by build.

Some operations share a schema; you'll find their operation IDs under its heading. Six calls in the catalog document HTTP 204 without a body. Some HTTP 200 schemas, including S029 and S086, don't say what the JSON looks like. This isn't a promise of full coverage or runtime compatibility.

### Schema S001

Used by: `fetchContent.response`.

```ts
type S001 = {
  "DisabledIDs": Array<unknown>;
  "Seasons": Array<{
      "ID": string;
      "Name": string;
      "Type": "episode" | "act";
      "StartTime": string;
      "EndTime": string;
      "IsActive": boolean;
    }>;
  "Events": Array<{
      "ID": string;
      "Name": string;
      "StartTime": string;
      "EndTime": string;
      "IsActive": boolean;
    }>;
};
```

### Schema S002

Used by: `accountXp.response`.

```ts
type S002 = {
  "Version": number;
  "Subject": string;
  "Progress": {
    "Level": number;
    "XP": number;
  };
  "History": Array<{
      "ID": string;
      "MatchStart": string;
      "StartProgress": {
        "Level": number;
        "XP": number;
      };
      "EndProgress": {
        "Level": number;
        "XP": number;
      };
      "XPDelta": number;
      "XPSources": Array<{
          "ID": "time-played" | "match-win" | "first-win-of-the-day";
          "Amount": number;
        }>;
      "XPMultipliers": Array<unknown>;
    }>;
  "LastTimeGrantedFirstWin": string;
  "NextTimeFirstWinAvailable": string;
};
```

### Schema S003

Used by: `playerLoadout.response`, `setPlayerLoadout.response`.

```ts
type S003 = {
  "Subject": string;
  "Version": number;
  "Guns": Array<{
      "ID": string;
      "CharmInstanceID"?: string;
      "CharmID"?: string;
      "CharmLevelID"?: string;
      "SkinID": string;
      "SkinLevelID": string;
      "ChromaID": string;
      "Attachments": Array<unknown>;
    }>;
  "ActiveExpressions": Array<{
      "TypeID": string;
      "AssetID": string;
    }>;
  "DynamicOptions": Record<string, unknown>;
  "Identity": {
    "PlayerCardID": string;
    "PlayerTitleID": string;
    "AccountLevel": number;
    "PreferredLevelBorderID": string;
    "HideAccountLevel": boolean;
  };
  "Incognito": boolean;
};
```

### Schema S004

Used by: `setPlayerLoadout.body`.

```ts
type S004 = {
  "Guns": Array<{
      "ID": string;
      "CharmInstanceID"?: string;
      "CharmID"?: string;
      "CharmLevelID"?: string;
      "SkinID": string;
      "SkinLevelID": string;
      "ChromaID": string;
      "Attachments": Array<unknown>;
    }>;
  "ActiveExpressions": Array<{
      "TypeID": string;
      "AssetID": string;
    }>;
  "DynamicOptions": Record<string, unknown>;
  "Identity": {
    "PlayerCardID": string;
    "PlayerTitleID": string;
    "AccountLevel": number;
    "PreferredLevelBorderID": string;
    "HideAccountLevel": boolean;
  };
  "Incognito": boolean;
};
```

### Schema S005

Used by: `playerMmr.response`.

```ts
type S005 = {
  "Version": number;
  "Subject": string;
  "NewPlayerExperienceFinished": boolean;
  "QueueSkills": Record<string, {
      "TotalGamesNeededForRating": number;
      "TotalGamesNeededForLeaderboard": number;
      "CurrentSeasonGamesNeededForRating": number;
      "SeasonalInfoBySeasonID": Record<string, {
          "SeasonID": string;
          "NumberOfWins": number;
          "NumberOfWinsWithPlacements": number;
          "NumberOfGames": number;
          "Rank": number;
          "CapstoneWins": number;
          "LeaderboardRank": number;
          "CompetitiveTier": number;
          "RankedRating": number;
          "WinsByTier": Record<string, number> | null;
          "GamesNeededForRating": number;
          "TotalWinsNeededForRank": number;
        }>;
    }>;
  "LatestCompetitiveUpdate": {
    "MatchID": string;
    "MapID": string;
    "SeasonID": string;
    "MatchStartTime": number;
    "TierAfterUpdate": number;
    "TierBeforeUpdate": number;
    "RankedRatingAfterUpdate": number;
    "RankedRatingBeforeUpdate": number;
    "RankedRatingEarned": number;
    "RankedRatingPerformanceBonus": number;
    "CompetitiveMovement": string;
    "AFKPenalty": number;
  };
  "IsLeaderboardAnonymized": boolean;
  "IsActRankBadgeHidden": boolean;
};
```

### Schema S006

Used by: `matchHistory.response`.

```ts
type S006 = {
  "Subject": string;
  "BeginIndex": number;
  "EndIndex": number;
  "Total": number;
  "History": Array<{
      "MatchID": string;
      "GameStartTime": number;
      "QueueID": string;
    }>;
};
```

### Schema S007

Used by: `matchDetails.response`.

```ts
type S007 = {
  "matchInfo": {
    "matchId": string;
    "mapId": string;
    "gamePodId": string;
    "gameLoopZone": string;
    "gameServerAddress": string;
    "gameVersion": string;
    "gameLengthMillis": number | null;
    "gameStartMillis": number;
    "provisioningFlowID": "Matchmaking" | "CustomGame";
    "isCompleted": boolean;
    "customGameName": string;
    "forcePostProcessing": boolean;
    "queueID": string;
    "gameMode": string;
    "isRanked": boolean;
    "isMatchSampled": boolean;
    "seasonId": string;
    "completionState": string;
    "platformType": "PC";
    "premierMatchInfo": object;
    "partyRRPenalties"?: Record<string, number>;
    "shouldMatchDisablePenalties": boolean;
  };
  "players": Array<{
      "subject": string;
      "gameName": string;
      "tagLine": string;
      "platformInfo": {
        "platformType": "PC";
        "platformOS": "Windows";
        "platformOSVersion": string;
        "platformChipset": "Unknown";
      };
      "teamId": "Blue" | "Red" | string;
      "partyId": string;
      "characterId": string;
      "stats": {
        "score": number;
        "roundsPlayed": number;
        "kills": number;
        "deaths": number;
        "assists": number;
        "playtimeMillis": number;
        "abilityCasts"?: {
          "grenadeCasts": number;
          "ability1Casts": number;
          "ability2Casts": number;
          "ultimateCasts": number;
        } | null;
      } | null;
      "roundDamage": Array<{
          "round": number;
          "receiver": string;
          "damage": number;
        }> | null;
      "competitiveTier": number;
      "isObserver": boolean;
      "playerCard": string;
      "playerTitle": string;
      "preferredLevelBorder"?: string | "";
      "accountLevel": number;
      "sessionPlaytimeMinutes"?: number | null;
      "xpModifications"?: Array<{
          "Value": number;
          "ID": string;
        }>;
      "behaviorFactors"?: {
        "afkRounds": number;
        "collisions"?: number;
        "commsRatingRecovery": number;
        "damageParticipationOutgoing": number;
        "friendlyFireIncoming"?: number;
        "friendlyFireOutgoing"?: number;
        "mouseMovement"?: number;
        "stayedInSpawnRounds"?: number;
      };
      "newPlayerExperienceDetails"?: {
        "basicMovement": {
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "basicGunSkill": {
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "adaptiveBots": {
          "adaptiveBotAverageDurationMillisAllAttempts": 0;
          "adaptiveBotAverageDurationMillisFirstAttempt": 0;
          "killDetailsFirstAttempt": null;
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "ability": {
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "bombPlant": {
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "defendBombSite": {
          "success": false;
          "idleTimeMillis": 0;
          "objectiveCompleteTimeMillis": 0;
        };
        "settingStatus": {
          "isMouseSensitivityDefault": boolean;
          "isCrosshairDefault": boolean;
        };
        "versionString": "";
      };
    }>;
  "bots": Array<unknown>;
  "coaches": Array<{
      "subject": string;
      "teamId": "Blue" | "Red";
    }>;
  "teams": Array<{
      "teamId": "Blue" | "Red" | string;
      "won": boolean;
      "roundsPlayed": number;
      "roundsWon": number;
      "numPoints": number;
    }> | null;
  "roundResults": Array<{
      "roundNum": number;
      "roundResult": "Eliminated" | "Bomb detonated" | "Bomb defused" | "Surrendered" | "Round timer expired";
      "roundCeremony": "CeremonyDefault" | "CeremonyTeamAce" | "CeremonyFlawless" | "CeremonyCloser" | "CeremonyClutch" | "CeremonyThrifty" | "CeremonyAce" | "";
      "winningTeam": "Blue" | "Red" | string;
      "bombPlanter"?: string;
      "bombDefuser"?: "Blue" | "Red" | string;
      "plantRoundTime"?: number;
      "plantPlayerLocations": Array<{
          "subject": string;
          "viewRadians": number;
          "location": {
            "x": number;
            "y": number;
          };
        }> | null;
      "plantLocation": {
        "x": number;
        "y": number;
      };
      "plantSite": "A" | "B" | "C" | "";
      "defuseRoundTime"?: number;
      "defusePlayerLocations": Array<{
          "subject": string;
          "viewRadians": number;
          "location": {
            "x": number;
            "y": number;
          };
        }> | null;
      "defuseLocation": {
        "x": number;
        "y": number;
      };
      "playerStats": Array<{
          "subject": string;
          "kills": Array<{
              "gameTime": number;
              "roundTime": number;
              "killer": string;
              "victim": string;
              "victimLocation": {
                "x": number;
                "y": number;
              };
              "assistants": Array<string>;
              "playerLocations": Array<{
                  "subject": string;
                  "viewRadians": number;
                  "location": {
                    "x": number;
                    "y": number;
                  };
                }>;
              "finishingDamage": {
                "damageType": "Weapon" | "Bomb" | "Ability" | "Fall" | "Melee" | "Invalid" | "";
                "damageItem": string | "Ultimate" | "Ability1" | "Ability2" | "GrenadeAbility" | "Primary" | "";
                "isSecondaryFireMode": boolean;
              };
            }>;
          "damage": Array<{
              "receiver": string;
              "damage": number;
              "legshots": number;
              "bodyshots": number;
              "headshots": number;
            }>;
          "score": number;
          "economy": {
            "loadoutValue": number;
            "weapon": string | "";
            "armor": string | "";
            "remaining": number;
            "spent": number;
          };
          "ability": {
            "grenadeEffects": null;
            "ability1Effects": null;
            "ability2Effects": null;
            "ultimateEffects": null;
          };
          "wasAfk": boolean;
          "wasPenalized": boolean;
          "stayedInSpawn": boolean;
        }>;
      "roundResultCode": "Elimination" | "Detonate" | "Defuse" | "Surrendered" | "";
      "playerEconomies": Array<{
          "subject": string;
          "loadoutValue": number;
          "weapon": string | "";
          "armor": string | "";
          "remaining": number;
          "spent": number;
        }> | null;
      "playerScores": Array<{
          "subject": string;
          "score": number;
        }> | null;
    }> | null;
  "kills": Array<{
      "gameTime": number;
      "roundTime": number;
      "killer": string;
      "victim": string;
      "victimLocation": {
        "x": number;
        "y": number;
      };
      "assistants": Array<string>;
      "playerLocations": Array<{
          "subject": string;
          "viewRadians": number;
          "location": {
            "x": number;
            "y": number;
          };
        }>;
      "finishingDamage": {
        "damageType": "Weapon" | "Bomb" | "Ability" | "Fall" | "Melee" | "Invalid" | "";
        "damageItem": string | "Ultimate" | "Ability1" | "Ability2" | "GrenadeAbility" | "Primary" | "";
        "isSecondaryFireMode": boolean;
      };
      "round": number;
    }> | null;
};
```

### Schema S008

Used by: `competitiveUpdates.response`.

```ts
type S008 = {
  "Version": number;
  "Subject": string;
  "Matches": Array<{
      "MatchID": string;
      "MapID": string;
      "SeasonID": string;
      "MatchStartTime": number;
      "TierAfterUpdate": number;
      "TierBeforeUpdate": number;
      "RankedRatingAfterUpdate": number;
      "RankedRatingBeforeUpdate": number;
      "RankedRatingEarned": number;
      "RankedRatingPerformanceBonus": number;
      "CompetitiveMovement": "MOVEMENT_UNKNOWN";
      "AFKPenalty": number;
    }>;
};
```

### Schema S009

Used by: `playerRegionIndex.response`.

```ts
type S009 = {
  "subject": string;
  "platformGroup": string;
  "seasonId": string;
  "regionData": Record<string, {
      "regionId": string;
      "values": {
        "winRate": number;
        "attackWinRate": number;
        "defenseWinRate": number;
        "tempValueF": number;
        "regionIndex": number;
        "totalGames": number;
        "computedFromValidSeasons": boolean;
      };
      "deltaValues": {
        "winRate": number;
        "attackWinRate": number;
        "defenseWinRate": number;
        "tempValueF": number;
        "regionIndex": number;
        "totalGames": number;
        "computedFromValidSeasons": boolean;
      };
      "bestMatch": {
        "matchId": string;
        "agentId": string;
        "actIndexBefore": number;
        "actIndexAfter": number;
        "tempValueF": number;
        "attackWinRate": number;
        "defenseWinRate": number;
        "outcome": "Win" | "Lose" | "Draw";
      };
      "matchHistory": Array<{
          "matchId": string;
          "agentId": string;
          "actIndexBefore": number;
          "actIndexAfter": number;
          "tempValueF": number;
          "attackWinRate": number;
          "defenseWinRate": number;
          "outcome": "Win" | "Lose" | "Draw";
        }>;
      "topAgents": Array<{
          "agentId": string;
        }>;
      "version": number;
    }>;
};
```

### Schema S010

Used by: `leaderboard.response`.

```ts
type S010 = {
  "Deployment": string;
  "QueueID": string;
  "SeasonID": string;
  "Players": Array<{
      "PlayerCardID": string;
      "TitleID": string;
      "IsBanned": boolean;
      "IsAnonymized": boolean;
      "puuid": string;
      "gameName": string;
      "tagLine": string;
      "leaderboardRank": number;
      "rankedRating": number;
      "numberOfWins": number;
      "competitiveTier": number;
    }>;
  "totalPlayers": number;
  "immortalStartingPage": number;
  "immortalStartingIndex": number;
  "topTierRRThreshold": number;
  "tierDetails": Record<string, {
      "rankedRatingThreshold": number;
      "startingPage": number;
      "startingIndex": number;
    }>;
  "startIndex": number;
  "query": string;
};
```

### Schema S011

Used by: `penalties.response`.

```ts
type S011 = {
  "Subject": string;
  "Penalties": Array<{
      "ID": string;
      "IssuingGameStartUnixMillis": number;
      "IssuingMatchID": string;
      "Expiry": string;
      "GamesRemaining": number;
      "ApplyToAllPlatforms": boolean;
      "ApplyToPlatforms": Array<string>;
      "ApplyToPlatformGroups": Array<string>;
      "InfractionID": string;
      "Origin": string;
      "ForgivenessIneligible": boolean;
      "IsAutomatedDetection": boolean;
      "PenaltyInfo"?: unknown | null;
      "DelayedPenaltyEffect"?: unknown | null;
      "GameBanEffect"?: unknown | null;
      "QueueDelayEffect"?: unknown | null;
      "QueueRestrictionEffect"?: unknown | null;
      "RankedRatingPenaltyEffect"?: unknown | null;
      "RiotRestrictionEffect"?: unknown | null;
      "RMSNotifyEffect"?: unknown | null;
      "WarningEffect": {
        "WarningType": string;
        "WarningTier": number;
        "ParentCentralRecordID": string;
      } | null;
      "XPMultiplierEffect"?: unknown | null;
      "PremierRestrictionEffect"?: unknown | null;
    }>;
  "Infractions"?: Array<{
      "ID": string;
      "Name": string;
      "RatingName": string;
    }>;
  "Version": number;
};
```

### Schema S012

Used by: `playerInterventions.response`.

```ts
type S012 = {
  "Subject": string;
  "InterventionsByCategory": Array<{
      "BehaviorCategory": string;
      "BehaviorRatingName": string;
      "LastRatingReduction": string;
      "ActiveInterventions": Array<{
          "InterventionName": string;
          "Expiry": string;
          "IssuingTime": string;
          "OriginInfraction": string;
        }>;
      "NextInterventionNames": Array<string>;
      "AppliedInfractions": Record<string, {
          "InfractionName": string;
          "Severity": string;
          "AppliedInterventions": Array<{
              "InterventionName": string;
              "Expiry": string;
              "IssuingTime": string;
              "OriginInfraction": string;
            }>;
        }>;
    }>;
};
```

### Schema S013

Used by: `playerReportToken.response`.

```ts
type S013 = {
  "Token": string;
};
```

### Schema S014

Used by: `playerAvoidList.response`.

```ts
type S014 = {
  "Subject": string;
  "AvoidList": Array<unknown>;
};
```

### Schema S015

Used by: `config.response`.

```ts
type S015 = {
  "LastApplication": string;
  "Collapsed": {
    "ARES_MOC_ENTITLEMENT": string;
    "CLIENT.ICONS.ENABLED": string;
    "CLIENT_LEADERBOARDS_ENABLED": string;
    "GAME_ALLOW_CONSOLE": string;
    "GAME_ALLOW_DEVELOPER_MENU": string;
    "GAME_DISABLED_DEATHCAM": string;
    "GAME_DISABLED_SKINS_WEAPONS": string;
    "GAME_PERFREPORTING_ENABLED": string;
    "GAME_REMOTE_MOVE_INTERP_ENABLED": string;
    "GAME_ROAMINGSETTINGS_ENABLED": string;
    "GAME_ROAMINGSETTINGS_KEY": string;
    "GAME_ROAMINGSETTINGS_STORAGEURL": string;
    "MAP_PRELOADING_ENABLED": string;
    "NAMECHECK_PLATFORM_REGION": string;
    "NAMECHECK_PLATFORM_URL": string;
    "ROSTER_REALM": string;
    "SECURITY_WATERMARK_ENABLED": string;
    "SECURITY_WATERMARK_MAX_OPACITY": string;
    "SECURITY_WATERMARK_MIN_OPACITY": string;
    "SECURITY_WATERMARK_TILING_FACTOR": string;
    "SERVICEURL_ACCOUNT_XP": string;
    "SERVICEURL_AGGSTATS": string;
    "SERVICEURL_AVS": string;
    "SERVICEURL_CONTENT": string;
    "SERVICEURL_CONTRACTS": string;
    "SERVICEURL_CONTRACT_DEFINITIONS": string;
    "SERVICEURL_COREGAME": string;
    "SERVICEURL_DAILY_TICKET": string;
    "SERVICEURL_FAVORITES": string;
    "SERVICEURL_GALBS_QUERY": string;
    "SERVICEURL_LATENCY": string;
    "SERVICEURL_LOGINQUEUE": string;
    "SERVICEURL_MASS_REWARDS": string;
    "SERVICEURL_MATCHDETAILS": string;
    "SERVICEURL_MATCHHISTORY": string;
    "SERVICEURL_MATCHMAKING": string;
    "SERVICEURL_MMR": string;
    "SERVICEURL_NAME": string;
    "SERVICEURL_PARTY": string;
    "SERVICEURL_PATCHNOTES": string;
    "SERVICEURL_PERSONALIZATION": string;
    "SERVICEURL_PLAYERFEEDBACK": string;
    "SERVICEURL_PREGAME": string;
    "SERVICEURL_PREMIER": string;
    "SERVICEURL_PROGRESSION": string;
    "SERVICEURL_PURCHASEMERCHANT": string;
    "SERVICEURL_REPLAY_CATALOG": string;
    "SERVICEURL_RESTRICTIONS": string;
    "SERVICEURL_SESSION": string;
    "SERVICEURL_STORE": string;
    "SERVICEURL_TOURNAMENTS": string;
    "SERVICE_TICKER_MESSAGE": string;
    "SERVICE_TICKER_MESSAGE.de-DE": string;
    "SERVICE_TICKER_MESSAGE.es-MX": string;
    "SERVICE_TICKER_MESSAGE.fr-FR": string;
    "SERVICE_TICKER_MESSAGE.it-IT": string;
    "SERVICE_TICKER_MESSAGE.pl-PL": string;
    "SERVICE_TICKER_MESSAGE.pt-BR": string;
    "SERVICE_TICKER_MESSAGE.ru-RU": string;
    "SERVICE_TICKER_MESSAGE.tr-TR": string;
    "SERVICE_TICKER_SEVERITY": string;
    "STORESCREEN_OFFERREFRESH_MAXDELAY_MILLISECONDS": string;
    "avs.enabled": string;
    "cap.location": string;
    "characterselect.debugwidgets.hide": string;
    "chat.mutedwords.enabled": string;
    "chat.v3.enabled": string;
    "collection.characters.enabled": string;
    "competitiveSeasonOffsetEndTime": string;
    "config.client.telemetry.samplerate": string;
    "content.filter.enabled": string;
    "content.maps.disabled": string;
    "eog.wip": string;
    "friends.enabled": string;
    "game.umgchat.enabled": string;
    "homescreen.featuredQueues": string;
    "homescreen.patchnotes.baseURL": string;
    "homescreen.promo.enabled": string;
    "homescreen.promo.key": string;
    "homescreen.webtile.baseURL": string;
    "loginqueue.region": string;
    "mainmenubar.collections.enabled": string;
    "mainmenubar.debug.enabled": string;
    "mainmenubar.profile.enabled": string;
    "mainmenubar.progression.enabled": string;
    "mainmenubar.shootingrange.enabled": string;
    "mainmenubar.store.enabled": string;
    "match.details.delay": string;
    "notifications.enabled": string;
    "parties.auto.balance.enabled": string;
    "party.observers.enabled": string;
    "partyinvites.enabled": string;
    "patchavailability.enabled": string;
    "personalization.equipAnyLevel.enabled": string;
    "personalization.useWidePlayerIdentityV2": string;
    "ping.update.interval": string;
    "ping.useGamePodsFromParties": string;
    "platformFaulted.level": string;
    "playerfeedbacktool.accessurl": string;
    "playerfeedbacktool.locale": string;
    "playerfeedbacktool.shard": string;
    "playerfeedbacktool.show": string;
    "playerfeedbacktool.survey_request_rate_float": string;
    "playscreen.partywidget.enabled": string;
    "playscreen.partywidget.matchmaking.maxsize": string;
    "playscreen.premier.enabled": string;
    "premier.conferences.fetch.enabled": string;
    "premier.leaderboardTab.enabled": string;
    "premier.matchHistoryTab.enabled": string;
    "premier.playscreenflow.enabled": string;
    "premier.rosterEligibilityCheck.enabled": string;
    "premier.seasons.activeseason.enabled": string;
    "premier.seasons.fetch.enabled": string;
    "queue.status.enabled": string;
    "rchat.ingame.enabled": string;
    "reporterfeedback.fetch.enabled": string;
    "reporterfeedback.notifications.enabled": string;
    "restrictions.v2.fetch.enabled": string;
    "restrictions.v2.warnings.enabled": string;
    "riotwarning.fetch.enabled": string;
    "riotwarning.notifications.enabled": string;
    "rnet.useAuthenticatedVoice": string;
    "russia.voice.enabled": string;
    "settings.livediagnostics.allowedplayers": string;
    "shootingtest.enabled": string;
    "skillrating.enabled": string;
    "skillrating.inGame.enabled": string;
    "skillrating.preGame.enabled": string;
    "social.panel.v6.enabled": string;
    "socialviewcontroller.enabled": string;
    "socialviewcontroller.v2.enabled": string;
    "store.isXgpDisabled": string;
    "store.use_currency_inventory_models": string;
    "store.use_platform_bundle_discounted_prices": string;
    "telemetry.rtp.eventendpoint": string;
    "telemetry.rtp.rfc190scope": string;
    "temp.voice.allowmuting": string;
    "tournaments.enabled": string;
    "tournaments.pregame.enabled": string;
    "vanguard.accessurl": string;
    "vanguard.netrequired": string;
    "voice.clutchmute.enabled": string;
    "voice.clutchmute.prompt.enabled": string;
    "voice.provider": string;
    "whisper.enabled": string;
  };
};
```

### Schema S016

Used by: `nameService.body`.

```ts
type S016 = Array<string>;
```

### Schema S017

Used by: `nameService.response`.

```ts
type S017 = Array<{
    "DisplayName": string;
    "Subject": string;
    "GameName": string;
    "TagLine": string;
  }>;
```

### Schema S018

Used by: `playerSession.response`.

```ts
type S018 = {
  "subject": string;
  "cxnState": string;
  "cxnCloseReason": string;
  "clientID": string;
  "clientVersion": string;
  "loopState": "MENUS" | "PREGAME" | "INGAME";
  "loopStateMetadata": string;
  "version": number;
  "lastHeartbeatTime": string;
  "expiredTime": string;
  "heartbeatIntervalMillis": number;
  "playtimeNotification": string;
  "playtimeMinutes": number;
  "isRestricted": boolean;
  "userinfoValidTime": string;
  "restrictionType": string;
  "clientPlatformInfo": {
    "platformType": string;
    "platformOS": string;
    "platformOSVersion": string;
    "platformChipset": string;
    "platformDevice": string;
  };
  "connectionTime": string;
  "shouldForceInvalidate": boolean;
};
```

### Schema S019

Used by: `party.response`, `partySetMemberReady.response`, `refreshCompetitiveTier.response`, `refreshPlayerIdentity.response`, `refreshPings.response`, `changeQueue.response`, `startCustomGame.response`, `enterMatchmakingQueue.response`, `leaveMatchmakingQueue.response`, `setPartyAccessibility.response`, `setCustomGameSettings.response`, `customGameMembership.response`, `setPlayerModeratorStatus.response`, `setPlayerBroadcastHudStatus.response`, `partyInvite.response`, `partyDecline.response`, `partyDisableCode.response`, `partyGenerateCode.response`.

```ts
type S019 = {
  "ID": string;
  "MUCName": string;
  "VoiceRoomID": string;
  "Version": number;
  "ClientVersion": string;
  "Members": Array<{
      "Subject": string;
      "CompetitiveTier": number;
      "PlayerIdentity": {
        "Subject": string;
        "PlayerCardID": string;
        "PlayerTitleID": string;
        "AccountLevel": number;
        "PreferredLevelBorderID": string | "";
        "Incognito": boolean;
        "HideAccountLevel": boolean;
      };
      "SeasonalBadgeInfo": null;
      "IsOwner"?: boolean;
      "QueueEligibleRemainingAccountLevels": number;
      "Pings": Array<{
          "Ping": number;
          "GamePodID": string;
        }>;
      "IsReady": boolean;
      "IsModerator": boolean;
      "UseBroadcastHUD": boolean;
      "PlatformType": "PC";
    }>;
  "State": string;
  "PreviousState": string;
  "StateTransitionReason": string;
  "Accessibility": "OPEN" | "CLOSED";
  "CustomGameData": {
    "Settings": {
      "Map": string;
      "Mode": string;
      "UseBots": boolean;
      "GamePod": string;
      "GameRules": {
        "AllowGameModifiers"?: string;
        "IsOvertimeWinByTwo"?: string;
        "PlayOutAllRounds"?: string;
        "SkipMatchHistory"?: string;
        "TournamentMode"?: string;
      } | null;
    };
    "Membership": {
      "teamOne": Array<{
          "Subject": string;
        }> | null;
      "teamTwo": Array<{
          "Subject": string;
        }> | null;
      "teamSpectate": Array<{
          "Subject": string;
        }> | null;
      "teamOneCoaches": Array<{
          "Subject": string;
        }> | null;
      "teamTwoCoaches": Array<{
          "Subject": string;
        }> | null;
    };
    "MaxPartySize": number;
    "AutobalanceEnabled": boolean;
    "AutobalanceMinPlayers": number;
    "HasRecoveryData": boolean;
  };
  "MatchmakingData": {
    "QueueID": string;
    "PreferredGamePods": Array<string>;
    "SkillDisparityRRPenalty": number;
  };
  "Invites": null;
  "Requests": Array<unknown>;
  "QueueEntryTime": string;
  "ErrorNotification": {
    "ErrorType": string;
    "ErroredPlayers": Array<{
        "Subject": string;
      }> | null;
  };
  "RestrictedSeconds": number;
  "EligibleQueues": Array<string>;
  "QueueIneligibilities": Array<string>;
  "CheatData": {
    "GamePodOverride": string;
    "ForcePostGameProcessing": boolean;
  };
  "XPBonuses": Array<unknown>;
  "InviteCode": string;
};
```

### Schema S020

Used by: `partyPlayer.response`, `startSoloExperience.response`, `partyJoinByCode.response`.

```ts
type S020 = {
  "Subject": string;
  "Version": number;
  "CurrentPartyID": string;
  "Invites": null;
  "Requests": Array<{
      "ID": string;
      "PartyID": string;
      "RequestedBySubject": string;
      "Subjects": Array<string>;
      "CreatedAt": string;
      "RefreshedAt": string;
      "ExpiresIn": number;
    }>;
  "PlatformInfo": {
    "platformType": "PC";
    "platformOS": "Windows";
    "platformOSVersion": string;
    "platformChipset": "Unknown";
  };
};
```

### Schema S021

Used by: `partySetMemberReady.body`.

```ts
type S021 = {
  "ready": boolean;
};
```

### Schema S022

Used by: `changeQueue.body`.

```ts
type S022 = {
  "queueId": string;
};
```

### Schema S023

Used by: `startSoloExperience.body`.

```ts
type S023 = {
  "gameType": "BotTrainingMatch" | "ShootingRange" | "NewPlayerExperience" | "WatchReplay" | "ReplayNewPlayerExperience";
  "module": string;
};
```

### Schema S024

Used by: `setPartyAccessibility.body`.

```ts
type S024 = {
  "accessibility": "OPEN" | "CLOSED";
};
```

### Schema S025

Used by: `setCustomGameSettings.body`.

```ts
type S025 = {
  "Map": string;
  "Mode": string;
  "UseBots": boolean;
  "GamePod": string;
  "GameRules": {
    "AllowGameModifiers": "true" | "false";
    "PlayOutAllRounds": "true" | "false";
    "SkipMatchHistory": "true" | "false";
    "TournamentMode": "true" | "false";
    "IsOvertimeWinByTwo": "true" | "false";
  };
};
```

### Schema S026

Used by: `customGameMembership.body`.

```ts
type S026 = {
  "playerToPutOnTeam": string;
};
```

### Schema S027

Used by: `setPlayerModeratorStatus.body`.

```ts
type S027 = {
  "playerToSetModeratorStatus": string;
  "moderatorStatus": boolean;
};
```

### Schema S028

Used by: `setPlayerBroadcastHudStatus.body`.

```ts
type S028 = {
  "playerToSetBroadcastHUDStatus": string;
  "broadcastHUDStatus": boolean;
};
```

### Schema S029

Used by: `partyRequest.response`, `premierRosterCustomization.response`.

```ts
type S029 = unknown;
```

### Schema S030

Used by: `customGameConfigs.response`.

```ts
type S030 = {
  "Enabled": boolean;
  "EnabledMaps": Array<string>;
  "EnabledModes": Array<string>;
  "Queues": Array<{
      "QueueID": string;
      "Enabled": boolean;
      "DebugOnDemandSweepEnabled": boolean;
      "TeamSize": number;
      "NumTeams": number;
      "MaxPartySize": number;
      "MinPartySize": number;
      "InvalidPartySizes": Array<number>;
      "MaxPartySizeHighSkill": number;
      "HighSkillTier": number;
      "MaxSkillTier": number;
      "AllowFullPartyBypassSkillRestrictions": boolean;
      "ApplyRRPenaltyToFullParty": boolean;
      "AllowFiveStackRestrictions": boolean;
      "Mode": string;
      "IsRanked": boolean;
      "IsTournament": boolean;
      "IsPatient": boolean;
      "RequireRoster": boolean;
      "Priority": number;
      "PartyMaxCompetitiveTierRange": number;
      "PartyMaxCompetitiveTierRangePlacementBuffer": number;
      "FullPartyMaxCompetitiveTierRange": number;
      "PartySkillDisparityCompetitiveTiersCeilings": Array<unknown> | Record<string, number>;
      "PartySkillDisparityPartySizeCompetitiveTiersCeilings": Array<unknown> | Record<string, Record<string, number>>;
      "NewMap": string;
      "NewMapEndTime": string;
      "NewMapLossReductionModifier": number;
      "SharedAccountsRequireVerificationEnabled": boolean;
      "HighRankRequireVerificationEnabled": boolean;
      "RequireVerificationEnabled": boolean;
      "UseAccountLevelRequirement": boolean;
      "MinimumAccountLevelRequired": number;
      "GameRules": Record<string, string>;
      "SupportedPlatformTypes": Array<"PC">;
      "DisabledContent": Array<unknown>;
      "DebugLoadoutAllowedContentWhenSkippingPregameEnabled": boolean;
      "DebugLoadoutAllowedContentWhenSkippingPregame": Array<unknown>;
      "queueFieldA": Array<unknown>;
      "NextScheduleChangeSeconds": number;
      "TimeUntilNextScheduleChangeSeconds": number;
      "MapWeights": Array<string>;
      "MapPool": Array<string>;
      "TeamSizeWithBots": number;
      "EnableMultiteamSupport": boolean;
      "MatchFillerMarsPartySizeBonus": number;
      "MatchFillerMarsAllowPartialTeams": boolean;
      "MatchFillerMarsMinTeams": number;
      "MatchFillerMarsMaxTeams": number;
      "MatchFillerMarsTeamAnchorStrategy": string;
      "MatchFillerMarsLowerTeamCount": number;
      "MatchFillerMarsLowerTeamCountMMR": number;
      "MatchFillerMarsUpperTeamCount": number;
      "MatchFillerMarsUpperTeamCountMMR": number;
      "queueFieldB": boolean;
    }>;
  "GamePodPingServiceInfo": Record<string, {
      "SecurityHash": number;
      "ObfuscatedIP": number;
      "PingProxyAddress": string;
      "PingProxyAddresses": Array<string>;
    }>;
};
```

### Schema S031

Used by: `partyChatToken.response`, `partyVoiceToken.response`.

```ts
type S031 = {
  "Token": string;
  "Room": string;
};
```

### Schema S032

Used by: `storefront.body`.

```ts
type S032 = object;
```

### Schema S033

Used by: `storefront.response`.

```ts
type S033 = {
  "FeaturedBundle": {
    "Bundle": {
      "ID": string;
      "DataAssetID": string;
      "CurrencyID": string;
      "Items": Array<{
          "Item": {
            "ItemTypeID": string;
            "ItemID": string;
            "Amount": number;
          };
          "BasePrice": number;
          "CurrencyID": string;
          "DiscountPercent": number;
          "DiscountedPrice": number;
          "IsPromoItem": boolean;
        }>;
      "ItemOffers": Array<{
          "BundleItemOfferID": string;
          "Offer": {
            "OfferID": string;
            "IsDirectPurchase": boolean;
            "StartDate": string;
            "Cost": Record<string, number>;
            "Rewards": Array<{
                "ItemTypeID": string;
                "ItemID": string;
                "Quantity": number;
              }>;
          };
          "DiscountPercent": number;
          "DiscountedCost": Record<string, number>;
        }> | null;
      "TotalBaseCost": Record<string, number> | null;
      "TotalDiscountedCost": Record<string, number> | null;
      "TotalDiscountPercent": number;
      "DurationRemainingInSeconds": number;
      "WholesaleOnly": boolean;
    };
    "Bundles": Array<{
        "ID": string;
        "DataAssetID": string;
        "CurrencyID": string;
        "Items": Array<{
            "Item": {
              "ItemTypeID": string;
              "ItemID": string;
              "Amount": number;
            };
            "BasePrice": number;
            "CurrencyID": string;
            "DiscountPercent": number;
            "DiscountedPrice": number;
            "IsPromoItem": boolean;
          }>;
        "ItemOffers": Array<{
            "BundleItemOfferID": string;
            "Offer": {
              "OfferID": string;
              "IsDirectPurchase": boolean;
              "StartDate": string;
              "Cost": Record<string, number>;
              "Rewards": Array<{
                  "ItemTypeID": string;
                  "ItemID": string;
                  "Quantity": number;
                }>;
            };
            "DiscountPercent": number;
            "DiscountedCost": Record<string, number>;
          }> | null;
        "TotalBaseCost": Record<string, number> | null;
        "TotalDiscountedCost": Record<string, number> | null;
        "TotalDiscountPercent": number;
        "DurationRemainingInSeconds": number;
        "WholesaleOnly": boolean;
      }>;
    "BundleRemainingDurationInSeconds": number;
  };
  "SkinsPanelLayout": {
    "SingleItemOffers": Array<string>;
    "SingleItemStoreOffers": Array<{
        "OfferID": string;
        "IsDirectPurchase": boolean;
        "StartDate": string;
        "Cost": Record<string, number>;
        "Rewards": Array<{
            "ItemTypeID": string;
            "ItemID": string;
            "Quantity": number;
          }>;
      }>;
    "SingleItemOffersRemainingDurationInSeconds": number;
  };
  "UpgradeCurrencyStore": {
    "UpgradeCurrencyOffers": Array<{
        "OfferID": string;
        "StorefrontItemID": string;
        "Offer": {
          "OfferID": string;
          "IsDirectPurchase": boolean;
          "StartDate": string;
          "Cost": Record<string, number>;
          "Rewards": Array<{
              "ItemTypeID": string;
              "ItemID": string;
              "Quantity": number;
            }>;
        };
        "DiscountedPercent": number;
      }>;
  };
  "AccessoryStore": {
    "AccessoryStoreOffers": Array<{
        "Offer": {
          "OfferID": string;
          "IsDirectPurchase": boolean;
          "StartDate": string;
          "Cost": Record<string, number>;
          "Rewards": Array<{
              "ItemTypeID": string;
              "ItemID": string;
              "Quantity": number;
            }>;
        };
        "ContractID": string;
      }>;
    "AccessoryStoreRemainingDurationInSeconds": number;
    "StorefrontID": string;
  };
  "PluginStores"?: Array<{
      "PluginID": string;
      "PluginInstanceID": string;
      "StorefrontItemID": string;
      "Offers": Array<{
          "OfferID": string;
          "StorefrontItemID": string;
          "StartDate": string;
          "EndDate": string;
          "Priority": number;
          "Cost": Record<string, number>;
          "Rewards": Array<{
              "ItemTypeID": string;
              "ItemID": string;
              "Quantity": number;
            }>;
        }>;
      "DurationRemainingInSeconds": number;
      "StorefrontExpiry": string;
    }>;
  "BonusStore"?: {
    "BonusStoreOffers": Array<{
        "BonusOfferID": string;
        "Offer": {
          "OfferID": string;
          "IsDirectPurchase": boolean;
          "StartDate": string;
          "Cost": Record<string, number>;
          "Rewards": Array<{
              "ItemTypeID": string;
              "ItemID": string;
              "Quantity": number;
            }>;
        };
        "DiscountPercent": number;
        "DiscountCosts": Record<string, number>;
        "IsSeen": boolean;
      }>;
    "BonusStoreRemainingDurationInSeconds": number;
  };
};
```

### Schema S034

Used by: `createOrder.body`.

```ts
type S034 = {
  "XID": string;
  "OfferID": string;
};
```

### Schema S035

Used by: `createOrder.response`.

```ts
type S035 = {
  "OrderID": string;
  "Status": "ACCEPTED" | "COMPLETE" | "FAILED";
};
```

### Schema S036

Used by: `getOrder.response`.

```ts
type S036 = {
  "OrderID": string;
  "Status": "ACCEPTED" | "COMPLETE" | "FAILED";
  "OrderRewards": Record<string, Array<{
        "RewardID": string;
        "Amount": number;
      }>>;
};
```

### Schema S037

Used by: `wallet.response`.

```ts
type S037 = {
  "Balances": Record<string, number>;
};
```

### Schema S038

Used by: `ownedItems.response`, `allOwnedItems.response`.

```ts
type S038 = {
  "EntitlementsByTypes": Array<{
      "ItemTypeID": string;
      "Entitlements": Array<{
          "TypeID": string;
          "ItemID": string;
          "InstanceID"?: string;
        }>;
    }>;
};
```

### Schema S039

Used by: `giftPurchaserEligibility.response`.

```ts
type S039 = {
  "FailureReasons": Array<string>;
  "GiftingEligibleStatus": number;
};
```

### Schema S040

Used by: `giftRecipientEligibility.response`.

```ts
type S040 = {
  "PurchaserFailureReasons": Array<string>;
  "RecipientFailureReasons": Array<string>;
  "RecipientOffers": Array<unknown>;
};
```

### Schema S041

Used by: `agentStorefront.response`.

```ts
type S041 = {
  "AgentStore": {
    "AgentStoreOffers": Array<{
        "AgentID": string;
        "StoreOffers": Array<{
            "OfferID": string;
            "IsDirectPurchase": boolean;
            "StartDate": string;
            "Cost": Record<string, number>;
            "Rewards": Array<{
                "ItemTypeID": string;
                "ItemID": string;
                "Quantity": number;
              }>;
          }>;
      }>;
    "CurrentFeaturedAgent": string;
    "NextFeaturedAgent": string;
  };
};
```

### Schema S042

Used by: `preGamePlayer.response`, `currentGamePlayer.response`.

```ts
type S042 = {
  "Subject": string;
  "MatchID": string;
  "Version": number;
};
```

### Schema S043

Used by: `preGameMatch.response`, `selectCharacter.response`, `lockCharacter.response`.

```ts
type S043 = {
  "ID": string;
  "Version": number;
  "Teams": Array<{
      "TeamID": "Blue" | "Red" | string;
      "Players": Array<{
          "Subject": string;
          "CharacterID": string;
          "CharacterSelectionState": "" | "selected" | "locked";
          "PregamePlayerState": string;
          "CompetitiveTier": number;
          "PlayerIdentity": {
            "Subject": string;
            "PlayerCardID": string;
            "PlayerTitleID": string;
            "AccountLevel": number;
            "PreferredLevelBorderID": string | "";
            "Incognito": boolean;
            "HideAccountLevel": boolean;
          };
          "SeasonalBadgeInfo": {
            "SeasonID": string | "";
            "NumberOfWins": number;
            "WinsByTier": null;
            "Rank": number;
            "LeaderboardRank": number;
          };
          "IsCaptain": boolean;
        }>;
    }>;
  "AllyTeam": {
    "TeamID": "Blue" | "Red" | string;
    "Players": Array<{
        "Subject": string;
        "CharacterID": string;
        "CharacterSelectionState": "" | "selected" | "locked";
        "PregamePlayerState": string;
        "CompetitiveTier": number;
        "PlayerIdentity": {
          "Subject": string;
          "PlayerCardID": string;
          "PlayerTitleID": string;
          "AccountLevel": number;
          "PreferredLevelBorderID": string | "";
          "Incognito": boolean;
          "HideAccountLevel": boolean;
        };
        "SeasonalBadgeInfo": {
          "SeasonID": string | "";
          "NumberOfWins": number;
          "WinsByTier": null;
          "Rank": number;
          "LeaderboardRank": number;
        };
        "IsCaptain": boolean;
      }>;
  } | null;
  "EnemyTeam": {
    "TeamID": "Blue" | "Red" | string;
    "Players": Array<{
        "Subject": string;
        "CharacterID": string;
        "CharacterSelectionState": "" | "selected" | "locked";
        "PregamePlayerState": string;
        "CompetitiveTier": number;
        "PlayerIdentity": {
          "Subject": string;
          "PlayerCardID": string;
          "PlayerTitleID": string;
          "AccountLevel": number;
          "PreferredLevelBorderID": string | "";
          "Incognito": boolean;
          "HideAccountLevel": boolean;
        };
        "SeasonalBadgeInfo": {
          "SeasonID": string | "";
          "NumberOfWins": number;
          "WinsByTier": null;
          "Rank": number;
          "LeaderboardRank": number;
        };
        "IsCaptain": boolean;
      }>;
  } | null;
  "ObserverSubjects": Array<unknown>;
  "MatchCoaches": Array<unknown>;
  "EnemyTeamSize": number;
  "EnemyTeamLockCount": number;
  "PregameState": string;
  "LastUpdated": string;
  "MapID": string;
  "MapSelectPool": Array<unknown>;
  "BannedMapIDs": Array<unknown>;
  "CastedVotes"?: unknown;
  "MapSelectSteps": Array<unknown>;
  "MapSelectStep": number;
  "Team1": "Blue" | "Red" | string;
  "GamePodID": string;
  "Mode": string;
  "VoiceSessionID": string;
  "MUCName": string;
  "TeamMatchToken": string;
  "QueueID": string | "";
  "ProvisioningFlowID": "Matchmaking" | "CustomGame";
  "IsRanked": boolean;
  "PhaseTimeRemainingNS": number;
  "StepTimeRemainingNS": number;
  "altModesFlagADA": boolean;
  "TournamentMetadata": null;
  "RosterMetadata": null;
};
```

### Schema S044

Used by: `preGameLoadouts.response`.

```ts
type S044 = {
  "Loadouts": Array<{
      "Subject": string;
      "Sprays": {
        "SpraySelections": Array<{
            "SocketID": string;
            "SprayID": string;
            "LevelID": string;
          }>;
      };
      "Expressions": {
        "AESSelections": Array<{
            "SocketID": string;
            "AssetID": string;
            "TypeID": string;
          }>;
      };
      "Items": Record<string, {
          "ID": string;
          "TypeID": string;
          "Sockets": Record<string, {
              "ID": string;
              "Item": {
                "ID": string;
                "TypeID": string;
              };
            }>;
        }>;
    }>;
  "LoadoutsValid": boolean;
};
```

### Schema S045

Used by: `currentGameMatch.response`.

```ts
type S045 = {
  "MatchID": string;
  "Version": number;
  "State": string;
  "MapID": string;
  "ModeID": string;
  "ProvisioningFlow": "Matchmaking" | "CustomGame";
  "GamePodID": string;
  "AllMUCName": string;
  "TeamMUCName": string;
  "TeamVoiceID": string;
  "TeamMatchToken": string;
  "IsReconnectable": boolean;
  "ConnectionDetails": {
    "GameServerHosts": Array<string>;
    "GameServerHost": string;
    "GameServerPort": number;
    "GameServerObfuscatedIP": number;
    "GameClientHash": number;
    "PlayerKey": string;
  };
  "PostGameDetails": null;
  "Players": Array<{
      "Subject": string;
      "TeamID": "Blue" | "Red" | string;
      "CharacterID": string;
      "PlayerIdentity": {
        "Subject": string;
        "PlayerCardID": string;
        "PlayerTitleID": string;
        "AccountLevel": number;
        "PreferredLevelBorderID": string | "";
        "Incognito": boolean;
        "HideAccountLevel": boolean;
      };
      "SeasonalBadgeInfo": {
        "SeasonID": string | "";
        "NumberOfWins": number;
        "WinsByTier": null;
        "Rank": number;
        "LeaderboardRank": number;
      };
      "IsCoach": boolean;
      "IsAssociated": boolean;
    }>;
  "MatchmakingData": null;
};
```

### Schema S046

Used by: `currentGameLoadouts.response`.

```ts
type S046 = {
  "Loadouts": Array<{
      "CharacterID": string;
      "Loadout": {
        "Subject": string;
        "Sprays": {
          "SpraySelections": Array<{
              "SocketID": string;
              "SprayID": string;
              "LevelID": string;
            }>;
        };
        "Expressions": {
          "AESSelections": Array<{
              "SocketID": string;
              "AssetID": string;
              "TypeID": string;
            }>;
        };
        "Items": Record<string, {
            "ID": string;
            "TypeID": string;
            "Sockets": Record<string, {
                "ID": string;
                "Item": {
                  "ID": string;
                  "TypeID": string;
                };
              }>;
          }>;
      };
    }>;
};
```

### Schema S047

Used by: `premierPlayer.response`.

```ts
type S047 = {
  "puuid": string;
  "rosterId": string;
  "invites": Array<unknown>;
  "version": number;
  "createdAt": number;
  "updatedAt": number;
};
```

### Schema S048

Used by: `premierPlayerCrests.response`.

```ts
type S048 = {
  "puuid": string;
  "contenderEligibilityExpiry": string;
  "seasons": Record<string, {
      "seasonId": string;
      "rosterId": string;
      "rosterName": string;
      "rosterTag": string;
      "conference": string;
      "division": number;
      "points": number;
      "crest": string;
      "championshipPointRequirement": number;
      "gamesPlayedByEventType": Record<string, number>;
    }>;
};
```

### Schema S049

Used by: `premierEligibility.response`.

```ts
type S049 = {
  "subject": string;
  "accountVerificationStatus": boolean;
  "rankedPlacementCompletionStatus": boolean;
  "matchLimitReached": boolean;
  "isOnProvidedRoster": boolean;
  "playerRestrictions"?: unknown | null;
};
```

### Schema S050

Used by: `premierSeasons.response`.

```ts
type S050 = {
  "PremierSeasons": Array<{
      "ID": string;
      "Name": string;
      "CompetitiveSeasonID": string;
      "PreviousPremierSeasonID": string;
      "NextPremierSeasonID": string;
      "IsActive": boolean;
      "StartTime": string;
      "EndTime": string;
      "Events": Array<{
          "ID": string;
          "Type": "LEAGUE" | "TOURNAMENT" | "SCRIM";
          "StartDateTime": string;
          "EndDateTime": string;
          "SchedulePerConference"?: unknown | null;
          "MapSelectionStrategy": "RANDOM" | "PICKBAN";
          "MapPoolMapIDs": Array<string>;
          "PointsRequiredToParticipate": number;
        }>;
      "ScheduledEvents": Array<unknown>;
      "Conferences": Array<{
          "id": string;
          "key": string;
          "isSuper": boolean;
          "gamePods": Array<string>;
          "timezone": string;
          "superConference": string;
          "leaderboardPlayoffQualificationDateTime": string;
          "leaderboardPromotionFinalizationDateTime": string;
        }>;
      "Divisions": Array<{
          "Division": number;
          "DivisionName": string;
          "DivisionGroup": string;
          "EventPresetName": string;
        }>;
      "DivisionThresholds": Array<unknown>;
      "EventPresets": Record<string, unknown>;
      "FlawlessPointRequirement": number;
      "ChampionshipPointRequirement": number;
      "ChampionshipEventID": string;
      "EnrollmentPhaseStartDateTime": string;
      "EnrollmentPhaseEndDateTime": string;
      "LeaderboardPlayoffQualificationDateTime": string;
      "LeaderboardPromotionFinalizationDateTime": string;
      "LeaderboardFinalizationDateTime": string;
      "ContenderEligibilityExpiryDateTime": string;
      "StageLockTime": string;
      "ParticipationRewardsActIDs": Array<string>;
      "ParticipationRewards"?: unknown;
      "TournamentWinnerRewards"?: unknown;
      "DivisionWinnerRewards"?: unknown;
    }>;
};
```

### Schema S051

Used by: `premierSeasonActive.response`.

```ts
type S051 = {
  "ID": string;
  "Name": string;
  "CompetitiveSeasonID": string;
  "PreviousPremierSeasonID": string;
  "NextPremierSeasonID": string;
  "IsActive": boolean;
  "StartTime": string;
  "EndTime": string;
  "Events": Array<{
      "ID": string;
      "Type": "LEAGUE" | "TOURNAMENT" | "SCRIM";
      "StartDateTime": string;
      "EndDateTime": string;
      "SchedulePerConference"?: unknown | null;
      "MapSelectionStrategy": "RANDOM" | "PICKBAN";
      "MapPoolMapIDs": Array<string>;
      "PointsRequiredToParticipate": number;
    }>;
  "ScheduledEvents": Array<unknown>;
  "Conferences": Array<{
      "id": string;
      "key": string;
      "isSuper": boolean;
      "gamePods": Array<string>;
      "timezone": string;
      "superConference": string;
      "leaderboardPlayoffQualificationDateTime": string;
      "leaderboardPromotionFinalizationDateTime": string;
    }>;
  "Divisions": Array<{
      "Division": number;
      "DivisionName": string;
      "DivisionGroup": string;
      "EventPresetName": string;
    }>;
  "DivisionThresholds": Array<unknown>;
  "EventPresets": Record<string, unknown>;
  "FlawlessPointRequirement": number;
  "ChampionshipPointRequirement": number;
  "ChampionshipEventID": string;
  "EnrollmentPhaseStartDateTime": string;
  "EnrollmentPhaseEndDateTime": string;
  "LeaderboardPlayoffQualificationDateTime": string;
  "LeaderboardPromotionFinalizationDateTime": string;
  "LeaderboardFinalizationDateTime": string;
  "ContenderEligibilityExpiryDateTime": string;
  "StageLockTime": string;
  "ParticipationRewardsActIDs": Array<string>;
  "ParticipationRewards"?: unknown;
  "TournamentWinnerRewards"?: unknown;
  "DivisionWinnerRewards"?: unknown;
};
```

### Schema S052

Used by: `premierConferences.response`.

```ts
type S052 = {
  "PremierConferences": Array<{
      "id": string;
      "key": string;
      "isSuper": boolean;
      "gamePods": Array<string>;
      "timezone": string;
      "superConference"?: string;
      "leaderboardPlayoffQualificationDateTime": string;
      "leaderboardPromotionFinalizationDateTime": string;
    }>;
};
```

### Schema S053

Used by: `upcomingMatches.response`.

```ts
type S053 = {
  "Leagues": Array<{
      "ID": string;
      "Name": string;
      "ImageURL": string;
      "TournamentIDs": Array<string>;
      "TeamIDs": Array<string> | null;
    }>;
  "Tournaments": Array<{
      "ID": string;
      "Name": string;
      "LeagueID": string;
      "LeagueName": string;
      "StartTime": string;
      "EndTime": string;
      "StageIDs": Array<string>;
      "TeamIDs": Array<string>;
    }>;
  "Teams": Array<{
      "ID": string;
      "Name": string;
      "Code": string;
      "BaseImageURL": string;
      "HighResImageURL": string;
      "LowResImageURL": string;
      "HomeLeagueID": string;
      "TeamMemberIDs": Array<string> | null;
      "BundleID": string;
      "BundleDataAssetID": string;
      "DataAssetID": string;
    }>;
  "Seasons": null;
  "Splits": null;
};
```

### Schema S054

Used by: `matches.body`.

```ts
type S054 = {
  "MATCHIDS": Array<string>;
};
```

### Schema S055

Used by: `matches.response`.

```ts
type S055 = {
  "Matches": Array<{
      "ID": string;
      "LeagueID": string;
      "TournamentID": string;
      "StageID": string;
      "StageName": string;
      "StructuralID": string;
      "StartTime": string;
      "State": string;
      "Destinations": Record<string, {
          "StructuralID": string;
          "Type": string;
          "Slot": number;
        }>;
      "MatchTeams": Array<{
          "TeamID": string;
          "OriginStructuralID": string;
          "OriginType": string;
          "OriginSlot": number;
          "Result": string;
          "GameWins": number;
        }>;
      "Streams": Array<unknown>;
    }>;
};
```

### Schema S056

Used by: `roster.response`.

```ts
type S056 = {
  "rosterId": string;
  "affinity": string;
  "name": string;
  "tag": string;
  "customization": {
    "icon": string;
    "primaryColor": string;
    "secondaryColor": string;
    "tertiaryColor": string;
  };
  "members": Array<{
      "puuid": string;
      "role": "OWNER" | "MEMBER";
      "roleId": number;
      "createdAt": number;
    }>;
  "invites": Array<unknown>;
  "locks": Array<unknown>;
  "season": {
    "id": string;
    "isEnrolled": boolean;
    "conference": string;
    "division": number;
    "isProvisionalDivision": boolean;
    "promotionApplied": boolean;
    "points": number;
    "wins": number;
    "gamesPlayed": number;
    "seasonMatchRoundWins": number;
    "seasonMatchRoundsPlayed": number;
    "crest": string;
    "hasEarnedPromotionForNextSeason": boolean;
    "hasEarnedPrestige": boolean;
  };
  "minimumRequiredMembersForEnrollment": number;
  "matchesSinceReset": number;
  "tournamentsSinceReset": number;
  "version": {
    "socialVersion": number;
    "premierVersion": number;
  };
  "updatedAt": number;
  "createdAt": number;
  "seasonalInfoBySeasonID": Record<string, {
      "id": string;
      "isEnrolled": boolean;
      "conference": string;
      "division": number;
      "isProvisionalDivision": boolean;
      "promotionApplied": boolean;
      "points": number;
      "wins": number;
      "gamesPlayed": number;
      "seasonMatchRoundWins": number;
      "seasonMatchRoundsPlayed": number;
      "crest": string;
      "hasEarnedPromotionForNextSeason": boolean;
      "hasEarnedPrestige": boolean;
    }>;
  "prestige": {
    "earnedPrestige": boolean;
    "plating": string;
  };
};
```

### Schema S057

Used by: `premierRosterMatchHistory.response`.

```ts
type S057 = {
  "id": string;
  "leagueMatchHistory": Array<{
      "matchId": string;
      "leaguePointsBefore": number;
      "leaguePointsAfter": number;
      "leaguePointsEarned": number;
      "startTime": number;
    }>;
  "tournamentMatchHistory": Array<{
      "tournamentId": string;
      "finalPlacement": number;
      "finalPlacementLeaguePointsBonus": number;
      "leaguePointsBefore": number;
      "leaguePointsAfter": number;
      "leaguePointsEarned": number;
      "startTime": number;
      "matchEntries": Record<string, number>;
      "tournamentMatchData": Record<string, {
          "points": number;
          "roundNumber": number;
          "totalRounds": number;
          "bracketType": string;
        }>;
    }>;
};
```

### Schema S058

Used by: `premierRosterPublicInfo.response`.

```ts
type S058 = {
  "rosterId": string;
  "name": string;
  "tag": string;
  "customization": {
    "icon": string;
    "primaryColor": string;
    "secondaryColor": string;
    "tertiaryColor": string;
  };
  "members": Array<{
      "puuid": string;
      "role": "OWNER" | "MEMBER";
      "roleId": number;
      "createdAt": number;
    }>;
  "season": {
    "id": string;
    "isEnrolled": boolean;
    "conference": string;
    "division": number;
    "isProvisionalDivision": boolean;
    "promotionApplied": boolean;
    "points": number;
    "wins": number;
    "gamesPlayed": number;
    "seasonMatchRoundWins": number;
    "seasonMatchRoundsPlayed": number;
    "crest": string;
    "plating": string;
    "matches"?: Record<string, {
        "matchId": string;
        "seasonId": string;
        "eventId": string;
        "startTime": number;
        "pointDeltas": {
          "before": number;
          "after": number;
          "earned": number;
        };
        "outcome": string;
        "opponentRosterID": string;
      }>;
    "tournaments"?: Record<string, {
        "tournamentId": string;
        "placement": number;
        "startTime": number;
        "pointDeltas": {
          "before": number;
          "after": number;
          "earned": number;
        };
        "matches": Record<string, {
            "points": number;
            "roundNumber": number;
            "totalRounds": number;
            "bracketType": string;
          }>;
      }>;
    "hasEarnedPromotionForNextSeason": boolean;
    "hasEarnedPrestige": boolean;
  };
  "version": {
    "socialVersion": number;
    "premierVersion": number;
  };
  "createdAt": number;
  "seasonalInfoBySeasonID": Record<string, {
      "id": string;
      "isEnrolled": boolean;
      "conference": string;
      "division": number;
      "isProvisionalDivision": boolean;
      "promotionApplied": boolean;
      "points": number;
      "wins": number;
      "gamesPlayed": number;
      "seasonMatchRoundWins": number;
      "seasonMatchRoundsPlayed": number;
      "crest": string;
      "plating": string;
      "matches"?: Record<string, {
          "matchId": string;
          "seasonId": string;
          "eventId": string;
          "startTime": number;
          "pointDeltas": {
            "before": number;
            "after": number;
            "earned": number;
          };
          "outcome": string;
          "opponentRosterID": string;
        }>;
      "tournaments"?: Record<string, {
          "tournamentId": string;
          "placement": number;
          "startTime": number;
          "pointDeltas": {
            "before": number;
            "after": number;
            "earned": number;
          };
          "matches": Record<string, {
              "points": number;
              "roundNumber": number;
              "totalRounds": number;
              "bracketType": string;
            }>;
        }>;
      "hasEarnedPromotionForNextSeason": boolean;
      "hasEarnedPrestige": boolean;
    }>;
};
```

### Schema S059

Used by: `premierRosterCustomization.body`.

```ts
type S059 = {
  "icon": string;
  "primaryColor": string;
  "secondaryColor": string;
  "tertiaryColor": string;
};
```

### Schema S060

Used by: `mailbox.response`.

```ts
type S060 = Array<{
    "mailId": string;
    "puuid": string;
    "message": string;
    "product": string;
    "region": string | null;
    "tags": Array<string>;
    "mailType": string;
    "state": string;
    "createdAt": number;
  }>;
```

### Schema S061

Used by: `mail.response`.

```ts
type S061 = {
  "mailId": string;
  "puuid": string;
  "message": string;
  "product": string;
  "region": string | null;
  "tags": Array<string>;
  "mailType": string;
  "state": string;
  "createdAt": number;
};
```

### Schema S062

Used by: `goldstars.response`.

```ts
type S062 = Array<{
    "id": string;
    "tempT": string;
  }>;
```

### Schema S063

Used by: `playerGoldstars.response`.

```ts
type S063 = {
  "puuid": string;
  "version": number;
  "tempS": Record<string, {
      "tempS": string;
      "tempA": Record<string, {
          "id": string;
          "tempC": number;
          "tempB": number;
        }>;
    }>;
  "tempM": Array<{
      "id": string;
      "tempG": number;
      "tempP": Record<string, {
          "tempA": Record<string, {
              "id": string;
              "tempV": number;
              "tempB": boolean;
            }>;
        }>;
    }>;
};
```

### Schema S064

Used by: `activityLookup.body`.

```ts
type S064 = {
  "product": "valorant";
  "party": string;
};
```

### Schema S065

Used by: `activityLookup.response`.

```ts
type S065 = {
  "activityId": string;
};
```

### Schema S066

Used by: `sharedJoinLink.response`.

```ts
type S066 = {
  "joinCode": string;
  "smartUrl": string;
  "expiresAt": string;
  "isActive": boolean;
};
```

### Schema S067

Used by: `bloomlinePlayer.response`, `bloomlinePlayerBonsai.response`, `updateBloomlinePlayerBonsai.response`.

```ts
type S067 = {
  "Subject": string;
  "Bonsais": Array<{
      "BonsaiID": string;
      "HypeID"?: string;
      "BackgroundID"?: string;
      "ClipID"?: string;
      "BorderID"?: string;
      "Flourish": {
        "FlourishID"?: string;
        "ShortLevel"?: number;
        "PreviousShortLevel"?: number;
        "LongLevel"?: number;
        "Points"?: number;
        "PointsVersion"?: number;
      };
      "NumberModules"?: Array<{
          "ModuleID"?: string;
          "NumberID": string;
          "Value": number;
        }>;
      "TrackProgressList"?: Array<{
          "TrackID": string;
          "TrackLevel"?: number;
        }>;
    }>;
  "LongGroupVersion"?: number;
};
```

### Schema S068

Used by: `updateBloomlinePlayerBonsai.body`.

```ts
type S068 = {
  "bonsaiId": string;
  "hypeId": string;
  "backgroundId": string;
  "clipId": string;
  "borderId": string;
  "moduleUpdates": Array<{
      "iD": string;
      "type": string;
      "item": string;
    }>;
};
```

### Schema S069

Used by: `bloomlineValidQueues.response`.

```ts
type S069 = {
  "ProcessedQueues": Array<string>;
  "PerformanceScoreQueues": Array<string>;
};
```

### Schema S070

Used by: `itemUpgrades.response`.

```ts
type S070 = {
  "Definitions": Array<{
      "ID": string;
      "Item": {
        "ItemTypeID": string;
        "ItemID": string;
      };
      "RequiredEntitlement": {
        "ItemTypeID": string;
        "ItemID": string;
      };
      "ProgressionSchedule": {
        "Name": string;
        "ProgressionCurrencyID": string;
        "ProgressionDeltaPerLevel": Array<number> | null;
      };
      "RewardSchedule": {
        "ID": string;
        "Name": string;
        "Prerequisites": null;
        "RewardsPerLevel": Array<{
            "EntitlementRewards": Array<{
                "Amount": number;
                "ItemTypeID": string;
                "ItemID": string;
              }>;
            "WalletRewards": null;
            "CounterRewards": null;
          }> | null;
      };
      "Sidegrades": Array<{
          "SidegradeID": string;
          "Options": Array<{
              "OptionID": string;
              "Cost": {
                "WalletCosts": Array<{
                    "CurrencyID": string;
                    "AmountToDeduct": number;
                  }>;
              };
              "Rewards": Array<{
                  "Amount": number;
                  "ItemTypeID": string;
                  "ItemID": string;
                }>;
            }>;
          "Prerequisites": {
            "RequiredEntitlements": Array<{
                "ItemTypeID": string;
                "ItemID": string;
              }>;
          };
        }> | null;
    }>;
};
```

### Schema S071

Used by: `contracts.response`, `activateContract.response`.

```ts
type S071 = {
  "Version": number;
  "Subject": string;
  "Contracts": Array<{
      "ContractDefinitionID": string;
      "ContractProgression": {
        "TotalProgressionEarned": number;
        "TotalProgressionEarnedVersion": number;
        "HighestRewardedLevel": Record<string, {
            "Amount": number;
            "Version": number;
          }>;
      };
      "ProgressionLevelReached": number;
      "ProgressionTowardsNextLevel": number;
    }>;
  "ProcessedMatches": Array<{
      "ID": string;
      "StartTime": number;
      "XPGrants": {
        "GamePlayed": number;
        "GameWon": number;
        "RoundPlayed": number;
        "RoundWon": number;
        "Missions": object;
        "Modifier": {
          "Value": number;
          "BaseMultiplierValue": number;
          "Modifiers": Array<{
              "Value": number;
              "Name": "RESTRICTIONS_XP" | "PREMIUM_CONTRACT_XP" | "SOCIAL_XP";
              "BaseOnly": boolean;
            }>;
        };
        "NumAFKRounds": number;
      } | null;
      "RewardGrants": object | null;
      "MissionDeltas": Record<string, {
          "ID": string;
          "Objectives": Record<string, number>;
          "ObjectiveDeltas": Record<string, {
              "ID": string;
              "ProgressBefore": number;
              "ProgressAfter": number;
            }>;
        }> | null;
      "ContractDeltas": Record<string, {
          "ID": string;
          "TotalXPBefore": number;
          "TotalXPAfter": number;
        }> | null;
      "CouldProgressMissions": boolean;
    }>;
  "ActiveSpecialContract": string;
  "Missions": Array<{
      "ID": string;
      "Objectives": Record<string, number>;
      "Complete": boolean;
      "ExpirationTime": string;
    }>;
  "MissionMetadata": {
    "NPECompleted": boolean;
    "WeeklyCheckpoint": string;
    "WeeklyRefillTime": string;
  };
};
```

### Schema S072

Used by: `contractMatch.response`.

```ts
type S072 = {
  "ID": string;
  "CreatedAt": string;
  "StartTime": number;
  "XPGrants": {
    "GamePlayed": number;
    "GameWon": number;
    "RoundPlayed": number;
    "RoundWon": number;
    "Missions": Record<string, number> | null;
    "Modifier": {
      "Value": number;
      "BaseMultiplierValue": number;
      "Modifiers": Array<{
          "Value": number;
          "Name": string;
          "BaseOnly": boolean;
        }>;
    };
    "NumAFKRounds": number;
  } | null;
  "DoughGrants": {
    "GamePlayed": number;
    "RoundPlayed": number;
    "RoundWon": number;
  } | null;
  "RewardGrants": Record<string, {
      "EntitlementRewards": Array<{
          "ItemTypeID": string;
          "ItemID": string;
          "Amount": number;
        }> | null;
      "ItemRewards": Array<unknown> | null;
      "WalletRewards": Array<{
          "CurrencyID": string;
          "Amount": number;
        }> | null;
      "CounterRewards": Array<unknown> | null;
      "ProgressionRewards": Array<unknown> | null;
    }> | null;
  "MissionDeltas": Record<string, {
      "ID": string;
      "Objectives": Record<string, number>;
      "ObjectiveDeltas": Record<string, {
          "ID": string;
          "ProgressBefore": number;
          "ProgressAfter": number;
        }>;
    }> | null;
  "ContractDeltas": Record<string, {
      "ID": string;
      "TotalXPBefore": number;
      "TotalXPAfter": number;
    }> | null;
  "CouldProgressMissions": boolean;
  "MatchSummary": {
    "RoundsTotal": number;
    "RoundsWon": number;
  };
  "XID": string;
};
```

### Schema S073

Used by: `localHelp.response`.

```ts
type S073 = {
  "events": Record<string, string>;
  "functions": Record<string, string>;
  "types": Record<string, string>;
};
```

### Schema S074

Used by: `sessions.response`.

```ts
type S074 = Record<string, {
    "exitCode": number;
    "exitReason": null;
    "isInternal": boolean;
    "launchConfiguration": {
      "arguments": Array<string>;
      "executable": string;
      "locale": string | null;
      "voiceLocale": null;
      "workingDirectory": string;
    };
    "patchlineFullName": "VALORANT" | "riot_client";
    "patchlineId": "" | "live" | "pbe";
    "phase": string;
    "productId": "valorant" | "riot_client";
    "version": string;
  }>;
```

### Schema S075

Used by: `rsoUserInfo.response`.

```ts
type S075 = {
  "userInfo": string;
};
```

### Schema S076

Used by: `clientRegion.response`.

```ts
type S076 = {
  "locale": string;
  "region": string;
  "webLanguage": string;
  "webRegion": string;
};
```

### Schema S077

Used by: `accountAlias.response`.

```ts
type S077 = {
  "active": boolean;
  "created_datetime": number;
  "game_name": string;
  "summoner": boolean;
  "tag_line": string;
};
```

### Schema S078

Used by: `entitlementsToken.response`.

```ts
type S078 = {
  "accessToken": string;
  "entitlements": Array<unknown>;
  "issuer": string;
  "subject": string;
  "token": string;
};
```

### Schema S079

Used by: `chatSession.response`.

```ts
type S079 = {
  "federated": boolean;
  "game_name": string;
  "game_tag": string;
  "loaded": boolean;
  "name": string;
  "pid": string;
  "puuid": string;
  "region": string;
  "resource": string;
  "state": string;
};
```

### Schema S080

Used by: `friends.response`.

```ts
type S080 = {
  "friends": Array<{
      "activePlatform": string | null;
      "displayGroup": string;
      "game_name": string;
      "game_tag": string;
      "group": string;
      "last_online_ts": number | null;
      "name": string;
      "note": string;
      "pid": string;
      "puuid": string;
      "region": string;
    }>;
};
```

### Schema S081

Used by: `sendFriendRequest.body`.

```ts
type S081 = {
  "game_name": string;
  "game_tag": string;
};
```

### Schema S082

Used by: `sendFriendRequest.response`.

```ts
type S082 = {
  "requests": Array<unknown>;
};
```

### Schema S083

Used by: `removeFriendRequest.body`.

```ts
type S083 = {
  "puuid": string;
};
```

### Schema S084

Used by: `friendRequests.response`.

```ts
type S084 = {
  "requests": Array<{
      "game_name": string;
      "game_tag": string;
      "name": string;
      "note": string;
      "pid": string;
      "puuid": string;
      "region": string;
      "subscription": "pending_out" | "pending_in";
    }>;
};
```

### Schema S085

Used by: `presence.response`.

```ts
type S085 = {
  "presences": Array<{
      "actor"?: unknown | null;
      "basic": string;
      "details"?: unknown | null;
      "game_name": string;
      "game_tag": string;
      "location"?: unknown | null;
      "msg"?: unknown | null;
      "name": string;
      "patchline"?: unknown | null;
      "pid": string;
      "platform"?: unknown | null;
      "private": string | null;
      "privateJwt"?: unknown | null;
      "product": "valorant" | "league_of_legends";
      "puuid": string;
      "region": string;
      "resource": string;
      "state": "mobile" | "dnd" | "away" | "chat";
      "summary": string;
      "time": number;
    }>;
};
```

### Schema S086

Used by: `localSwaggerDocs.response`.

```ts
type S086 = unknown;
```

### Schema S087

Used by: `partyChatInfo.response`, `preGameChatInfo.response`, `currentGameChatInfo.response`, `allChatInfo.response`.

```ts
type S087 = {
  "conversations": Array<{
      "cid": string;
      "direct_messages": boolean;
      "global_readership": boolean;
      "message_history": boolean;
      "mid": string;
      "muted": boolean;
      "mutedRestriction": boolean;
      "type": "groupchat" | "chat";
      "uiState": {
        "changedSinceHidden": boolean;
        "hidden": boolean;
      };
      "unread_count": number;
    }>;
};
```

### Schema S088

Used by: `chatParticipants.response`.

```ts
type S088 = {
  "participants": Array<{
      "activePlatform": null;
      "cid": string;
      "game_name": string;
      "game_tag": string;
      "muted": boolean;
      "name": string;
      "pid": string;
      "puuid": string;
      "region": string;
    }>;
};
```

### Schema S089

Used by: `sendChat.body`.

```ts
type S089 = {
  "cid": string;
  "message": string;
  "type": "groupchat" | "chat" | "system";
};
```

### Schema S090

Used by: `sendChat.response`, `chatHistory.response`.

```ts
type S090 = {
  "messages": Array<{
      "body": string;
      "cid": string;
      "game_name": string;
      "game_tag": string;
      "id": string;
      "mid": string;
      "name": string;
      "pid": string;
      "puuid": string;
      "read": boolean;
      "region": string;
      "time": string;
      "type": "chat" | "groupchat";
    }>;
};
```
