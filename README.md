# routerra-api
Repository for Routerra API documentation and client side code

About us: https://routerra.io
Email: info@routerra.io
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

## 📤 Response

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

## 📤 Response

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
