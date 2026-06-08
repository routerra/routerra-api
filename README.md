# routerra-api
Repository for Routerra API documentation and client side code

About us: https://routerra.io
Email: info@routerra.io

---

# Route Optimization API Reference

**🔗 Base URL**  
```
https://api.routerra.io/external/v1
```

**🔐 Authentication**  
Include your API key in every request header:  
```
API-KEY: <your-key>
```

---

## Optimize a multi-stop route.

### 📤 Request

```
POST /optimize
API-KEY: <your-key>
Content-Type: application/json
```

**Body:**
```json
{
  "startLocation":      { … },
  "startTime":          "HH:mm",
  "date":               "yyyy-MM-dd",
  "stops":              [ … ],
  "finishLocation":     { … },
  "optimizeSettings":   { … },
  "saveRoute":          false,
  "routeName":          "My Route"
}
```

#### Request Fields

| Field               | Type                                 | Required | Description                                                      |
|---------------------|--------------------------------------|:--------:|------------------------------------------------------------------|
| `startLocation`     | `Location`                           |   yes    | Starting point for the route                                     |
| `finishLocation`    | `Location` \| `null`                 |    no    | Optional final drop-off location                                 |
| `stops`             | `Stop[]`                             |   yes    | List of intermediate stops to visit                              |
| `date`              | `string` (`yyyy-MM-dd`)              |    no    | Departure date. Defaults to current date if not provided         |
| `startTime`         | `string` (`HH:mm`)                   |   yes    | Departure time in local 24h format                               |
| `optimizeSettings`  | `OptimizationSettings`               |    no    | Optional optimization parameters. Defaults used if not provided  |
| `saveRoute`         | `boolean`                            |    no    | If `true`, saves route to database and returns shared link. Default: false |
| `routeName`         | `string` \| `null`                   |    no    | Name for the saved route. Only used when `saveRoute` is true     |

#### Location

| Field      | Type                   | Required | Description                      |
|------------|------------------------|:--------:|----------------------------------|
| `latitude` | `number`               |   yes    | WGS84 latitude (decimal degrees) |
| `longitude`| `number`               |   yes    | WGS84 longitude (decimal degrees)|
| `address`  | `string` \| `null`     |    no    | Optional human-readable address  |

#### Stop

| Field              | Type                                         | Required | Description                                         |
|--------------------|----------------------------------------------|:--------:|-----------------------------------------------------|
| `id`               | `long`                                       |   yes    | Unique identifier to match stops in response        |
| `location`         | `Location`                                   |   yes    | Coordinates and optional address for this stop      |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`                 |    no    | Earliest desired arrival time                       |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`                 |    no    | Latest desired arrival time                         |
| `serviceTime`      | `number` \| `null`                           |    no    | Service duration at stop, in seconds. Default: 0    |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"`       |    no    | AUTO: flexible; EARLIEST/LATEST: hard time windows. Default: AUTO |
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`             |    no    | Preferred side of road to stop. Default: ANY        |

#### OptimizationSettings

| Field            | Type                                                      | Required | Description                                              |
|------------------|-----------------------------------------------------------|:--------:|----------------------------------------------------------|
| `avoidTolls`     | `boolean`                                                 |    no    | If `true`, route avoids toll roads. Default: false       |
| `liveRoadData`   | `boolean`                                                 |    no    | If `true`, uses real-time traffic data. Default: false   |
| `avoidHighway`   | `boolean`                                                 |    no    | If `true`, route avoids highways. Default: false         |
| `vehicleType`    | `"BIKE"` \| `"SCOOTER"` \| `"CAR"` \| `"VAN"` \| `"TRUCK"` |    no    | Vehicle profile for routing. Default: CAR                |
| `optimizeBy`     | `"distance"` \| `"time"`                                  |    no    | Optimization goal: minimize distance or time. Default: distance |

---

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/optimize" \
  -H "API-KEY: your-key" \
  -H "Content-Type: application/json" \
  -d '{
        "startLocation": {
          "latitude": 52.2297,
          "longitude": 21.0122,
          "address": "Warsaw, Poland"
        },
        "date": "2025-01-20",
        "startTime": "08:00",
        "stops": [
          {
            "id": 1234,
            "location": {
              "latitude": 52.4064,
              "longitude": 16.9252,
              "address": "Poznan, Poland"
            },
            "arrivalRangeFrom": null,
            "arrivalRangeTo": null,
            "serviceTime": 600,
            "priority": "AUTO",
            "stopSide": "ANY"
          }
        ],
        "finishLocation": null,
        "optimizeSettings": {
          "avoidTolls": true,
          "liveRoadData": true,
          "avoidHighway": false,
          "vehicleType": "CAR",
          "optimizeBy": "distance"
        }
      }'
```

---

### 📤 Response

```json
{
  "statistics":             { … },
  "startLocation":          { … },
  "startLocationDeparture": "2025-01-20T08:00",
  "optimizedStops":         [ … ],
  "unassignedStops":        [ … ],
  "finishLocation":         { … },
  "finishDriveTime":        1800,
  "finishDriveDistance":    25000,
  "finishLocationArrival":  "2025-01-20T17:30",
  "routeId":                12345,
  "sharedLink":             "https://app.routerra.io/shared/abc123..."
}
```

#### Response Fields

| Field                    | Type                              | Description                                           |
|--------------------------|-----------------------------------|-------------------------------------------------------|
| `statistics`             | `RouteStatistics`                 | Aggregated route metrics                              |
| `startLocation`          | `Location`                        | Echo of the request start location                    |
| `startLocationDeparture` | `string` (`yyyy-MM-dd'T'HH:mm`)   | Departure datetime from start location                |
| `optimizedStops`         | `OptimizedStop[]`                 | Array of stops in optimized order                     |
| `unassignedStops`        | `OptimizedStop[]`                 | Array of stops that could not be assigned             |
| `finishLocation`         | `Location` \| `null`              | Echo of request finish location or `null`             |
| `finishDriveTime`        | `number` \| `null`                | Travel time from last stop to finish, in seconds      |
| `finishDriveDistance`    | `number` \| `null`                | Distance from last stop to finish, in meters          |
| `finishLocationArrival`  | `string` (`yyyy-MM-dd'T'HH:mm`)   | Expected arrival datetime at finish location          |
| `routeId`                | `number` \| `null`                | ID of saved route (only when `saveRoute=true`)        |
| `sharedLink`             | `string` \| `null`                | URL to access saved route (only when `saveRoute=true`)|

#### RouteStatistics

| Field       | Type     | Description                            |
|-------------|----------|----------------------------------------|
| `distance`  | `number` | Total distance in kilometers           |
| `time`      | `number` | Total drive + service time in seconds  |
| `stops`     | `number` | Total number of stops                  |

#### OptimizedStop

| Field             | Type                            | Description                                         |
|-------------------|---------------------------------|-----------------------------------------------------|
| `id`              | `long`                          | Identifier matching the request stop                |
| `position`        | `number`                        | Sequence index in optimized route (1 = first stop)  |
| `location`        | `Location`                      | Stop coordinates and address                        |
| `waitTime`        | `number` \| `null`              | Idle time waiting for time window, in seconds       |
| `driveTime`       | `number`                        | Travel time from previous point, in seconds         |
| `driveDistance`   | `number`                        | Travel distance from previous point, in meters      |
| `stopErrorType`   | `string` \| `null`              | Stop-level error, if any (see Error Types below)    |
| `expectedArrival` | `string` (`yyyy-MM-dd'T'HH:mm`) | Predicted arrival datetime at this stop             |

---

### ⚠️ Error Handling

#### Stop Error Types (`stopErrorType`)

| Error                      | Description                                      |
|----------------------------|--------------------------------------------------|
| `CANT_VISIT_TIME_WINDOW`   | Time window constraints can't be met             |
| `OUTSIDE_TRANSIT_AREA`     | Stop is outside routable area                    |
| `GEOCODE_FAILED`           | Failed to geocode the stop location              |
| `CAPACITY_EXCEEDED`        | Vehicle capacity exceeded                        |
| `NO_DRIVER_WITH_ZONE_ACCESS` | Stop is in a zone no driver can access         |

---

### 💡 Tips

- **Time formats**: Request uses `HH:mm`, response uses `yyyy-MM-dd'T'HH:mm`
- **Units**: Distances in km (statistics) or meters (per-stop); times in seconds
- **Unassigned stops**: Check `unassignedStops` array for stops that couldn't be optimized
- **Priority**: `EARLIEST`/`LATEST` enforce hard time windows; `AUTO` is flexible
- **Save Route**: Set `saveRoute: true` to persist the optimized route and get a shareable link. The saved route can be accessed via the web app or exported using the File Export API with the returned `routeId`.

---

# Route Management API Reference

Manage routes and stops programmatically. Create routes, add/update/remove stops, and calculate directions — all via the API.

---

## Create a route

### 📤 Request

```
POST /routes
API-KEY: <your-key>
Content-Type: application/json
```

**Body:**
```json
{
  "name":             "Monday deliveries",
  "date":             "2025-01-20",
  "startLocation":    { … },
  "startTime":        "08:00",
  "finishLocation":   { … },
  "optimizeSettings": { … }
}
```

#### Request Fields

| Field              | Type                    | Required | Description                                                  |
|--------------------|-------------------------|:--------:|--------------------------------------------------------------|
| `name`             | `string`                |    no    | Human-readable name for the route                            |
| `date`             | `string` (`yyyy-MM-dd`) |    no    | Route date                                                   |
| `startLocation`    | `Location`              |    no    | Starting point for the route                                 |
| `startTime`        | `string` (`HH:mm`)      |    no    | Departure time in local 24h format                           |
| `finishLocation`   | `Location`              |    no    | Optional return/end location                                 |
| `optimizeSettings` | `OptimizationSettings`  |    no    | Optimization parameters (see [OptimizationSettings](#optimizationsettings)) |

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/routes" \
  -H "API-KEY: your-key" \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Monday deliveries",
        "date": "2025-01-20",
        "startLocation": {
          "latitude": 52.2297,
          "longitude": 21.0122,
          "address": "Warsaw, Poland"
        },
        "startTime": "08:00",
        "finishLocation": null,
        "optimizeSettings": {
          "avoidTolls": false,
          "liveRoadData": true,
          "avoidHighway": false,
          "vehicleType": "CAR",
          "optimizeBy": "distance"
        }
      }'
```

### 📤 Response

Returns a [RouteResponse](#routeresponse) object.

---

## Get a route

### 📤 Request

```
GET /routes/{routeId}
API-KEY: <your-key>
```

#### Path Parameters

| Parameter | Type   | Required | Description          |
|-----------|--------|:--------:|----------------------|
| `routeId` | `long` |   yes    | The ID of the route  |

### 📦 Sample Request (cURL)

```bash
curl "https://api.routerra.io/external/v1/routes/12345" \
  -H "API-KEY: your-key"
```

### 📤 Response

Returns a [RouteResponse](#routeresponse) object with its stops.

---

## Add a stop to a route

### 📤 Request

```
POST /routes/{routeId}/stops
API-KEY: <your-key>
Content-Type: application/json
```

#### Path Parameters

| Parameter | Type   | Required | Description          |
|-----------|--------|:--------:|----------------------|
| `routeId` | `long` |   yes    | The ID of the route  |

**Body:**
```json
{
  "location": {
    "latitude": 52.4064,
    "longitude": 16.9252,
    "address": "Poznan, Poland"
  },
  "note":             "Ring doorbell",
  "arrivalRangeFrom": "09:00",
  "arrivalRangeTo":   "12:00",
  "serviceTime":      300,
  "load":             2,
  "position":         3,
  "priority":         "AUTO",
  "stopSide":         "ANY"
}
```

#### Request Fields

| Field              | Type                                         | Required | Description                                         |
|--------------------|----------------------------------------------|:--------:|-----------------------------------------------------|
| `location`         | `Location`                                   |   yes    | Coordinates and optional address for this stop      |
| `note`             | `string` \| `null`                           |    no    | Free-text note for this stop                        |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`                 |    no    | Earliest desired arrival time                       |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`                 |    no    | Latest desired arrival time                         |
| `serviceTime`      | `number` \| `null`                           |    no    | Service duration at stop, in seconds. Default: 0    |
| `load`             | `number` \| `null`                           |    no    | Load units for capacity constraints                 |
| `position`         | `number` \| `null`                           |    no    | Insert at this position (1-based). Existing stops at or after this position are shifted down. Default: appended at the end |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"`       |    no    | Time window enforcement mode. Default: AUTO         |
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`             |    no    | Preferred side of road to stop. Default: ANY        |

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/routes/12345/stops" \
  -H "API-KEY: your-key" \
  -H "Content-Type: application/json" \
  -d '{
        "location": {
          "latitude": 52.4064,
          "longitude": 16.9252,
          "address": "Poznan, Poland"
        },
        "serviceTime": 300,
        "priority": "AUTO",
        "stopSide": "ANY"
      }'
```

### 📤 Response

Returns a [StopResponse](#stopresponse) object.

---

## Update a stop

### 📤 Request

```
PUT /routes/{routeId}/stops/{stopId}
API-KEY: <your-key>
Content-Type: application/json
```

#### Path Parameters

| Parameter | Type   | Required | Description          |
|-----------|--------|:--------:|----------------------|
| `routeId` | `long` |   yes    | The ID of the route  |
| `stopId`  | `long` |   yes    | The ID of the stop   |

**Body:** Only include fields you want to update. Omitted fields remain unchanged.

```json
{
  "location": {
    "latitude": 52.41,
    "longitude": 16.93,
    "address": "Updated address"
  },
  "note": "Updated note",
  "serviceTime": 600
}
```

#### Request Fields

| Field              | Type                                         | Required | Description                                         |
|--------------------|----------------------------------------------|:--------:|-----------------------------------------------------|
| `location`         | `Location`                                   |    no    | Updated coordinates and/or address                  |
| `note`             | `string` \| `null`                           |    no    | Updated note                                        |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`                 |    no    | Updated earliest arrival time                       |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`                 |    no    | Updated latest arrival time                         |
| `serviceTime`      | `number` \| `null`                           |    no    | Updated service duration in seconds                 |
| `load`             | `number` \| `null`                           |    no    | Updated load units                                  |
| `position`         | `number` \| `null`                           |    no    | Move stop to this position (1-based). Other stops are shifted accordingly. Clamped to valid range |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"`       |    no    | Updated priority                                    |
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`             |    no    | Updated side-of-road preference                     |

### 📦 Sample Request (cURL)

```bash
curl -X PUT "https://api.routerra.io/external/v1/routes/12345/stops/67890" \
  -H "API-KEY: your-key" \
  -H "Content-Type: application/json" \
  -d '{
        "note": "Leave at front door",
        "serviceTime": 600
      }'
```

### 📤 Response

Returns a [StopResponse](#stopresponse) object.

---

## Delete a stop

### 📤 Request

```
DELETE /routes/{routeId}/stops/{stopId}
API-KEY: <your-key>
```

#### Path Parameters

| Parameter | Type   | Required | Description          |
|-----------|--------|:--------:|----------------------|
| `routeId` | `long` |   yes    | The ID of the route  |
| `stopId`  | `long` |   yes    | The ID of the stop   |

### 📦 Sample Request (cURL)

```bash
curl -X DELETE "https://api.routerra.io/external/v1/routes/12345/stops/67890" \
  -H "API-KEY: your-key"
```

### 📤 Response

Returns `200 OK` with no body.

---

## Calculate route directions

Calculates directions and statistics for a route based on its current stops. The stop order is preserved as-is (no reordering).

### 📤 Request

```
POST /routes/{routeId}/calculate
API-KEY: <your-key>
```

#### Path Parameters

| Parameter | Type   | Required | Description          |
|-----------|--------|:--------:|----------------------|
| `routeId` | `long` |   yes    | The ID of the route  |

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/routes/12345/calculate" \
  -H "API-KEY: your-key"
```

### 📤 Response

Returns a [RouteResponse](#routeresponse) object with calculated statistics, drive times, and distances.

---

## Route Management Response Types

### RouteResponse

```json
{
  "id":                    12345,
  "name":                  "Monday deliveries",
  "date":                  "2025-01-20",
  "status":                "ORDER_PRESERVED",
  "statistics":            { "distance": 125.4, "time": 7200, "stops": 5 },
  "startLocation":         { "latitude": 52.2297, "longitude": 21.0122, "address": "Warsaw" },
  "startTime":             "08:00",
  "finishLocation":        { "latitude": 52.2297, "longitude": 21.0122, "address": "Warsaw" },
  "finishDriveTime":       1800,
  "finishDriveDistance":   25000,
  "finishLocationArrival": "2025-01-20T17:30",
  "optimizeSettings":      { … },
  "stops":                 [ … ]
}
```

| Field                  | Type                             | Description                                              |
|------------------------|----------------------------------|----------------------------------------------------------|
| `id`                   | `number`                         | Route ID                                                 |
| `name`                 | `string` \| `null`               | Route name                                               |
| `date`                 | `string` (`yyyy-MM-dd`)          | Route date                                               |
| `status`               | `string`                         | Route status (e.g. `CREATED`, `ORDER_PRESERVED`)         |
| `statistics`           | `RouteStatistics`                | Aggregated route metrics (see [RouteStatistics](#routestatistics)) |
| `startLocation`        | `Location` \| `null`             | Start location                                           |
| `startTime`            | `string` (`HH:mm`) \| `null`    | Departure time                                           |
| `finishLocation`       | `Location` \| `null`             | Finish location                                          |
| `finishDriveTime`      | `number` \| `null`               | Travel time from last stop to finish, in seconds         |
| `finishDriveDistance`   | `number` \| `null`               | Distance from last stop to finish, in meters             |
| `finishLocationArrival`| `string` (`yyyy-MM-dd'T'HH:mm`) \| `null` | Expected arrival at finish location            |
| `optimizeSettings`     | `OptimizationSettings` \| `null` | Route optimization settings                              |
| `stops`                | `StopResponse[]`                 | List of stops on the route                               |

### StopResponse

```json
{
  "id":               67890,
  "position":         1,
  "location":         { "latitude": 52.4064, "longitude": 16.9252, "address": "Poznan" },
  "note":             "Ring doorbell",
  "arrivalRangeFrom": "09:00",
  "arrivalRangeTo":   "12:00",
  "serviceTime":      300,
  "load":             2,
  "priority":         "AUTO",
  "stopSide":         "ANY",
  "status":           "PENDING",
  "errorType":        null,
  "expectedArrival":  "2025-01-20T09:45",
  "waitTime":         null,
  "driveTime":        3600,
  "driveDistance":     30000
}
```

| Field             | Type                             | Description                                         |
|-------------------|----------------------------------|-----------------------------------------------------|
| `id`              | `number`                         | Stop ID                                             |
| `position`        | `number`                         | Sequence position in the route (1 = first stop)     |
| `location`        | `Location`                       | Stop coordinates and address                        |
| `note`            | `string` \| `null`               | Free-text note                                      |
| `arrivalRangeFrom`| `string` (`HH:mm`) \| `null`    | Earliest desired arrival time                       |
| `arrivalRangeTo`  | `string` (`HH:mm`) \| `null`    | Latest desired arrival time                         |
| `serviceTime`     | `number` \| `null`               | Service duration in seconds                         |
| `load`            | `number` \| `null`               | Load units                                          |
| `priority`        | `string`                         | `AUTO`, `EARLIEST`, or `LATEST`                     |
| `stopSide`        | `string`                         | `ANY`, `LEFT`, or `RIGHT`                           |
| `status`          | `string`                         | Stop status (e.g. `PENDING`, `COMPLETED`)           |
| `errorType`       | `string` \| `null`               | Stop-level error (see [Stop Error Types](#stop-error-types-stoperrortype)) |
| `expectedArrival` | `string` (`yyyy-MM-dd'T'HH:mm`) \| `null` | Predicted arrival datetime                |
| `waitTime`        | `number` \| `null`               | Idle time waiting for time window, in seconds       |
| `driveTime`       | `number` \| `null`               | Travel time from previous point, in seconds         |
| `driveDistance`    | `number` \| `null`               | Travel distance from previous point, in meters      |

---

# File Export API Reference

## Generate a temporary download link for exporting a route.

### 📤 Request

```
POST /routes/{id}/download-link/{fileType}
API-KEY: <your-key>
```

#### Path Parameters

| Parameter  | Type     | Required | Description                              |
|------------|----------|:--------:|------------------------------------------|
| `id`       | `long`   |   yes    | The ID of the route to export            |
| `fileType` | `string` |   yes    | Export format: `xlsx`, `csv`, or `pdf`   |

---

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/routes/12345/download-link/xlsx" \
  -H "API-KEY: your-key"
```

---

### 📤 Response

```json
{
  "downloadUrl": "https://routerra-exports.s3.amazonaws.com/exports/...",
  "expiresAt": 1706745600000,
  "filename": "My Route.xlsx",
  "fileType": "XLSX"
}
```

#### Response Fields

| Field         | Type     | Description                                     |
|---------------|----------|-------------------------------------------------|
| `downloadUrl` | `string` | Pre-signed URL to download the exported file    |
| `expiresAt`   | `number` | URL expiration timestamp (Unix milliseconds)    |
| `filename`    | `string` | Suggested filename for the download             |
| `fileType`    | `string` | Export format: `XLSX`, `CSV`, or `PDF`          |

---

### 📁 Supported File Types

| Type   | Content-Type                                                       | Description                     |
|--------|--------------------------------------------------------------------|---------------------------------|
| `xlsx` | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`| Microsoft Excel spreadsheet     |
| `csv`  | `text/csv`                                                         | Comma-separated values          |
| `pdf`  | `application/pdf`                                                  | PDF document with route details |

---

### 💡 Tips

- **Link Expiration**: Download URLs are temporary and expire after a set period (check `expiresAt`).
- **Single Use**: Generate a new link for each download session.
- **Large Routes**: For routes with many stops, prefer `xlsx` or `csv` for better performance.
