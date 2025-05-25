# routerra-api
Repository for Routerra API documentation and client side code

About us: https://routerra.io
Email: info@routerra.io
# 🚀 Route Optimization API Reference

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

## 📥 POST /optimize

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
  "startLocation":      { … },
  "startTime":          "HH:mm",
  "stops":              [ … ],
  "finishLocation":     { … } | null,
  "optimizeSettings":   { … }
}
```

#### Request Fields

| Icon | Field               | Type                                 | Required | Description                                                      |
|------|---------------------|--------------------------------------|:--------:|------------------------------------------------------------------|
| 📍   | **startLocation**   | `Location`                           |   yes    | Starting point for the route                                     |
| 🕒   | **startTime**       | `string` (`HH:mm`)                   |   yes    | Departure time in local 24h format                               |
| 🛑   | **stops**           | `Stop[]`                             |   yes    | List of intermediate stops to visit                              |
| 🏁   | **finishLocation**  | `Location` \| `null`                 |    no    | Optional final drop-off location                                 |
| ⚙️    | **optimizeSettings**| `OptimizationSettings`               |   yes    | Global optimization parameters                                   |

#### Location

| Field      | Type                   | Required | Description                      |
|------------|------------------------|:--------:|----------------------------------|
| `latitude` | `number`               |   yes    | WGS84 latitude (decimal degrees) |
| `longitude`| `number`               |   yes    | WGS84 longitude (decimal degrees)|
| `address`  | `string` \| `null`     |    no    | Optional human-readable address  |

#### Stop

| Field              | Type                                         | Required | Description                                         |
|--------------------|----------------------------------------------|:--------:|-----------------------------------------------------|
| `location`         | `Location`                                   |   yes    | Coordinates and optional address for this stop      |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`                 |    no    | Earliest desired arrival time                       |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`                 |    no    | Latest desired arrival time                         |
| `serviceTime`      | `number` \| `null`                           |    no    | Service duration at stop, in seconds                |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"`       |   yes    | AUTO: flexible; EARLIEST / LATEST: hard time windows|
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`             |   yes    | Preferred side of road to stop                      |

#### OptimizationSettings

| Field            | Type                                                   | Required | Description                                                       |
|------------------|--------------------------------------------------------|:--------:|-------------------------------------------------------------------|
| `avoidTolls`     | `boolean`                                              |   yes    | If `true`, route avoids toll roads                                |
| `liveRoadData`   | `boolean`                                              |   yes    | If `true`, uses real-time traffic data                            |
| `avoidHighway`   | `boolean`                                              |   yes    | If `true`, route avoids highways                                  |
| `vehicleType`    | `"BIKE"` \| `"SCOOTER"` \| `"CAR"` \| `"VAN"` \| `"TRUCK"` | yes | Vehicle profile for routing                                        |
| `optimizeBy`     | `"distance"` \| `"time"`                               |   yes    | Primary optimization goal: minimize total distance or time        |
| `serviceTime`    | `number`                                               |   yes    | Default service time per stop, in seconds                         |
| `stopSide`       | `"ANY"` \| `"LEFT"` \| `"RIGHT"`                       |   yes    | Default stop side preference                                      |

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
        "startTime": "08:00",
        "stops": [
          {
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
          "optimizeBy": "distance",
          "serviceTime": 300,
          "stopSide": "ANY"
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

| Icon | Field                   | Type                         | Description                                                       |
|------|-------------------------|------------------------------|-------------------------------------------------------------------|
| ⚠️   | `routeErrorType`        | `null` \| `string`           | High-level route error, if any                                    |
| 📊   | `statistics`            | `RouteStatistics`            | Aggregated route metrics                                          |
| 📍   | `startLocation`         | `Location`                   | Echo of the request start location                                |
| 🛑   | `stops`                 | `OptimizedStop[]`            | Array of planned stops in optimized order                         |
| 🏁   | `finishLocation`        | `Location` \| `null`         | Echo of request finish location or `null`                         |
| ⏱️   | `finishDriveTime`       | `null` \| `number`           | Travel time from last stop to finish, in seconds                  |
| 🛣️   | `finishDriveDistance`   | `null` \| `number`           | Distance from last stop to finish, in meters                      |
| ⏳   | `finishLocationArrival` | `string` (`HH:mm`)           | Expected arrival time at finish location                          |

#### RouteStatistics

| Field       | Type     | Description                            |
|-------------|----------|----------------------------------------|
| `distance`  | `number` | Total distance in kilometers           |
| `time`      | `number` | Total drive + service time in seconds  |
| `stops`     | `number` | Total number of stops                  |

#### OptimizedStop

| Field              | Type                        | Description                                                       |
|--------------------|-----------------------------|-------------------------------------------------------------------|
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

