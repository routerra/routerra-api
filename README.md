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

## POST /optimize

Optimize a multi-stop route.

### 📤 Request

```
POST /external/v1/optimize HTTP/1.1
Host: api.routerra.io
API-KEY: your-key
Content-Type: application/json
```

```
{
  "id":                 
  "startLocation":      { … },
  "startTime":          "HH:mm",
  "stops":              [ … ],
  "finishLocation":     { … } | null,
  "optimizeSettings":   { … }
}
```

#### Request Fields

| Field               | Type                                 | Required | Description                                                      |
|---------------------|--------------------------------------|:--------:|------------------------------------------------------------------|
| `startLocation`   | `Location`                           |   yes    | Starting point for the route                                     |
| `finishLocation`  | `Location` \| `null`                 |    no    | Optional final drop-off location                                 |
| `stops`           | `Stop[]`                             |   yes    | List of intermediate stops to visit                              |
| `date`            | `string` (`yyyy-MM-dd`)              |   no     | Optional Departure date in yyyy-MM-dd format. By default current date is used.|
| `startTime`       | `string` (`HH:mm`)                   |   yes    | Departure time in local 24h format                               |
| `optimizeSettings`| `OptimizationSettings`               |   no    | Optional global optimization parameters. Default parameters will be used if none provided. |

#### Location

| Field      | Type                   | Required | Description                      |
|------------|------------------------|:--------:|----------------------------------|
| `latitude` | `number`               |   yes    | WGS84 latitude (decimal degrees) |
| `longitude`| `number`               |   yes    | WGS84 longitude (decimal degrees)|
| `address`  | `string` \| `null`     |    no    | Optional human-readable address  |

#### Stop

| Field              | Type                                         | Required | Description                                         |
|--------------------|----------------------------------------------|:--------:|-----------------------------------------------------|
| `id`               | `long`                                       |   yes    | Identifier number for stop location which can be used to match stops in optimized route response |
| `location`         | `Location`                                   |   yes    | Coordinates and optional address for this stop      |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`                 |    no    | Earliest desired arrival time                       |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`                 |    no    | Latest desired arrival time                         |
| `serviceTime`      | `number` \| `null`                           |    no    | Service duration at stop, in seconds. Default 0s if not set. |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"`       |    no    | AUTO: flexible; EARLIEST / LATEST: hard time windows. Default: AUTO if not set. |
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`             |    no    | Preferred side of road to stop. Default: ANY if not set.                      |

#### OptimizationSettings

| Field            | Type                                                   | Required | Description                                                       |
|------------------|--------------------------------------------------------|:--------:|-------------------------------------------------------------------|
| `avoidTolls`     | `boolean`                                              |   no    | If `true`, route avoids toll roads. Default: false                                |
| `liveRoadData`   | `boolean`                                              |   no    | If `true`, uses real-time traffic data. Default: false                            |
| `avoidHighway`   | `boolean`                                              |   no    | If `true`, route avoids highways. Default: false                                   |
| `vehicleType`    | `"BIKE"` \| `"SCOOTER"` \| `"CAR"` \| `"VAN"` \| `"TRUCK"` | no | Vehicle profile for routing. Default: CAR                                        |
| `optimizeBy`     | `"distance"` \| `"time"`                               |   no    | Primary optimization goal: minimize total distance or time. Default: distance        |

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
        "date": ""
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

```
{
  "routeErrorType":       null | string,
  "statistics":           { … },
  "startLocation":        { … },
  "stops":                [ … ],
  "finishLocation":       { … } | null,
  "finishDriveTime":      null | number,
  "finishDriveDistance":  null | number,
  "finishLocationArrival":"HH:mm"
}
```

#### Response Fields

| Field                   | Type                         | Description                                                       |
|-------------------------|------------------------------|-------------------------------------------------------------------|
| `routeErrorType`        | `null` \| `string`           | High-level route error, if any                                    |
| `statistics`            | `RouteStatistics`            | Aggregated route metrics                                          |
| `startLocation`         | `Location`                   | Echo of the request start location                                |
| `stops`                 | `OptimizedStop[]`            | Array of planned stops in optimized order                         |
| `finishLocation`        | `Location` \| `null`         | Echo of request finish location or `null`                         |
| `finishDriveTime`       | `null` \| `number`           | Travel time from last stop to finish, in seconds                  |
| `finishDriveDistance`   | `null` \| `number`           | Distance from last stop to finish, in meters                      |
| `finishLocationArrival` | `string` (`HH:mm`)           | Expected arrival time at finish location                          |

#### RouteStatistics

| Field       | Type     | Description                            |
|-------------|----------|----------------------------------------|
| `distance`  | `number` | Total distance in kilometers           |
| `time`      | `number` | Total drive + service time in seconds  |
| `stops`     | `number` | Total number of stops                  |

#### OptimizedStop

| Field              | Type                        | Description                                                       |
|--------------------|-----------------------------|-------------------------------------------------------------------|
| `id`               | `long`                      | Identifier number for stop location which can be used to match stops in optimized route response |
| `position`         | `number`                    | Sequence index in optimized route (1 = first stop)               |
| `location`         | `Location`                  | Stop coordinates and address                                      |
| `waitTime`         | `null` \| `number`          | Idle time waiting for time window, in seconds                     |
| `driveTime`        | `number`                    | Travel time from previous point, in seconds                       |
| `driveDistance`    | `number`                    | Travel distance from previous point, in meters                    |
| `stopErrorType`    | `null` \| `string`          | Stop-level error, if any                                          |
| `expectedArrival`  | `string` (`HH:mm`)          | Predicted arrival time at this stop                               |

---

### ⚠️ Error Handling

Error responses include:

```
{
  "error": {
    "code":    "string",
    "message": "string"
  }
}
```

#### Error Types

**stopErrorType**  
- 🕑 `CANT_VISIT_TIME_WINDOW` — Time window constraints can’t be met  
- 📍 `OUTSIDE_TRANSIT_AREA` — Stop is outside routing area  
- ⚠️ `INVALID_TIME_WINDOW` — Provided time window invalid

**routeErrorType**  
- 📐 `MATRIX_CALCULATION_FAILED` — Distance/time matrix calculation failed  
- 🚧 `ROUTE_OPTIMIZATION_FAILED` — Optimization algorithm failure  
- 📊 `TOO_MANY_LOCATIONS_FOR_UNKNOWN_REGION` — Too many stops in undefined region  

---

### 💡 Tips

- **Time formats**: `HH:mm` (24-hour).  
- **Units**: Distances in km/meters; times in seconds.  
- **Defaults**: Per-stop `serviceTime` uses global default if omitted.  
- **Priority**: `EARLIEST`/`LATEST` enforce hard windows; `AUTO` is flexible.  


# File Export API Reference

## 📥 POST /routes/{routeId}/export-link/{fileType}

Generate a temporary download link for exporting a route in various formats.

### 📤 Request

```
POST /external/v1/routes/{routeId}/export-link/{fileType} HTTP/1.1
Host: api.routerra.io
API-KEY: your-key
```

#### Path Parameters

| Parameter | Type | Required | Description |
|------------|----------|:--------:|--------------------------------------------------|
| `routeId` | `long` | yes | The ID of the route to export |
| `fileType` | `string` | yes | Export format: `xlsx`, `csv`, or `pdf` |

---

### 📦 Sample Request (cURL)

```bash
curl -X POST "https://api.routerra.io/external/v1/routes/12345/export-link/xlsx" \
  -H "API-KEY: your-key"
```

---

## 📤 Response

```json
{
  "downloadUrl": "https://storage.routerra.io/exports/...",
  "expiresAt": 1706745600000,
  "filename": "My Route.xlsx",
  "fileType": "XLSX"
}
```

#### Response Fields

| Field | Type | Description |
|---------------|----------|-------------------------------------------------------------------|
| `downloadUrl` | `string` | Pre-signed URL to download the exported file |
| `expiresAt` | `number` | URL expiration timestamp (Unix milliseconds) |
| `filename` | `string` | Suggested filename for the download |
| `fileType` | `string` | Export format: `XLSX`, `CSV`, or `PDF` |

---

### 📁 Supported File Types

| Type | Content-Type | Description |
|--------|----------------------------------------|----------------------------------------|
| `xlsx` | `application/vnd.openxmlformats-...` | Microsoft Excel spreadsheet |
| `csv` | `text/csv` | Comma-separated values |
| `pdf` | `application/pdf` | PDF document with route details |

---

### 💡 Tips

- **Link Expiration**: Download URLs are temporary and expire after a set period (check `expiresAt`).
- **Single Use**: Generate a new link for each download session.
- **Large Routes**: For routes with many stops, prefer `xlsx` or `csv` for better performance.

