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
  "date":               "yyyy-MM-dd",
  "stops":              [ … ],
  "finishLocation":     { … } | null,
  "optimizeSettings":   { … }
}
```

#### Request Fields

| Icon | Field               | Type                                  | Required | Default        | Description                                               |
|------|---------------------|---------------------------------------|:--------:|:--------------:|-----------------------------------------------------------|
| 📍   | **startLocation**   | `Location`                            |   yes    |    -           | Starting point for the route                              |
| 🕒   | **startTime**       | `string` (`HH:mm`)                    |   yes    |    -           | Departure time in local 24h format                        |
| 📅   | **date**            | `string` (`yyyy-MM-dd`)               |    no    | Local date     | Planned date of the route                                 |
| 🛑   | **stops**           | `Stop[]`                              |   yes    |    -           | List of intermediate stops to visit                       |
| 🏁   | **finishLocation**  | `Location` \| `null`                  |    no    |    -           | Optional final drop-off location                          |
| ⚙️   | **optimizeSettings**| `OptimizationSettings`                |   yes    |    -           | Global optimization parameters                            |

#### Location

| Field      | Type                   | Required | Description                      |
|------------|------------------------|:--------:|----------------------------------|
| `latitude` | `number`               |   yes    | WGS84 latitude (decimal degrees) |
| `longitude`| `number`               |   yes    | WGS84 longitude (decimal degrees)|
| `address`  | `string` \| `null`     |    no    | Optional human-readable address  |

#### Stop

| Field              | Type                                   | Required | Default      | Description                                          |
|--------------------|----------------------------------------|:--------:|:------------:|------------------------------------------------------|
| `id`               | `number`                               |   yes    |    -         | Unique identifier of the stop                        |
| `location`         | `Location`                             |   yes    |    -         | Coordinates and optional address for this stop       |
| `arrivalRangeFrom` | `string` (`HH:mm`) \| `null`           |    no    |    -         | Earliest desired arrival time                        |
| `arrivalRangeTo`   | `string` (`HH:mm`) \| `null`           |    no    |    -         | Latest desired arrival time                          |
| `serviceTime`      | `number` \| `null`                     |    no    |   `0`        | Service duration at stop, in seconds                 |
| `priority`         | `"AUTO"` \| `"EARLIEST"` \| `"LATEST"` |    no    |   `"AUTO"`   | AUTO: flexible; EARLIEST / LATEST: hard time windows |
| `stopSide`         | `"ANY"` \| `"LEFT"` \| `"RIGHT"`       |    no    |   `"ANY"`    | Preferred side of road to stop                       |

#### OptimizationSettings

| Field            | Type                                                       | Default         | Description                                                |
|------------------|------------------------------------------------------------|:---------------:|------------------------------------------------------------|
| `avoidTolls`     | `boolean`                                                  |   `false`       | If `true`, route avoids toll roads                         |
| `liveRoadData`   | `boolean`                                                  |   `false`       | If `true`, uses real-time traffic data                     |
| `avoidHighway`   | `boolean`                                                  |   `false`       | If `true`, route avoids highways                           |
| `vehicleType`    | `"BIKE"` \| `"SCOOTER"` \| `"CAR"` \| `"VAN"` \| `"TRUCK"` |   `"CAR"`       | Vehicle profile for routing                                |
| `optimizeBy`     | `"DISTANCE"` \| `"TIME"`                                   |   `"DISTANCE"`  | Primary optimization goal: minimize total distance or time |

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
        "date": "2025-06-01",
        "stops": [
          {
            "id": 1,
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
          "optimizeBy": "DISTANCE"
        }
      }'
```

---

## 📤 Response

```
{
  "statistics":              { … },
  "startLocation":           { … },
  "startLocationDeparture": "yyyy-MM-dd'T'HH:mm",
  "optimizedStops":          [ … ],
  "unassignedStops":         [ … ],
  "finishLocation":          { … } | null,
  "finishDriveTime":         null | number,
  "finishDriveDistance":     null | number,
  "finishLocationArrival":   "yyyy-MM-dd'T'HH:mm"
}
```

#### Response Fields

| Icon | Field                   | Type                                       | Description                                                       |
|------|-------------------------|--------------------------------------------|-------------------------------------------------------------------|
| 📊   | `statistics`            | `RouteStatistics`                          | Aggregated route metrics                                          |
| 📍   | `startLocation`         | `Location`                                 | Echo of the request start location                                |
| 🕒   | `startLocationDeparture`| `string` (`yyyy-MM-dd'T'HH:mm`)            | Planned departure time from the starting location                 |
| 🛑   | `optimizedStops`        | `OptimizedStop[]`                          | Array of planned stops in optimized order                         |
| 🚫   | `unassignedStops`       | `OptimizedStop[]`                          | Array of stops unassigned due to constraints or errors            |
| 🏁   | `finishLocation`        | `Location` \| `null`                       | Echo of request finish location or `null`                         |
| ⏱️   | `finishDriveTime`       | `null` \| `number`                         | Travel time from last stop to finish, in seconds                  |
| 🛣️   | `finishDriveDistance`   | `null` \| `number`                         | Distance from last stop to finish, in meters                      |
| ⏳   | `finishLocationArrival` | `null` \| `string` (`yyyy-MM-dd'T'HH:mm`)  | Expected arrival time at finish location                          |

#### RouteStatistics

| Field       | Type     | Description                            |
|-------------|----------|----------------------------------------|
| `distance`  | `number` | Total distance in kilometers           |
| `time`      | `number` | Total drive + service time in seconds  |
| `stops`     | `number` | Total number of stops                  |

#### OptimizedStop

| Field              | Type                            | Description                                        |
|--------------------|---------------------------------|----------------------------------------------------|
| `id`               | `number`                        | Echo of unique identifier of the stop              |
| `position`         | `number`                        | Sequence index in optimized route (1 = first stop) |
| `location`         | `Location`                      | Stop coordinates and address                       |
| `waitTime`         | `null` \| `number`              | Idle time waiting for time window, in seconds      |
| `driveTime`        | `number`                        | Travel time from previous point, in seconds        |
| `driveDistance`    | `number`                        | Travel distance from previous point, in meters     |
| `stopErrorType`    | `null` \| `string`              | Stop-level error, if any                           |
| `expectedArrival`  | `string` (`yyyy-MM-dd'T'HH:mm`) | Predicted arrival time at this stop                |

---

### ⚠️ Error Handling

Error responses include:

```
{
    "type": "string",
    "title": "string",
    "status": number,
    "detail": "string",
    "instance": "string",
    "errors": []
}
```

#### Error Types

**stopErrorType**  
- 🕑 `CANT_VISIT_TIME_WINDOW` — Time window constraints can’t be met  
- 📍 `OUTSIDE_TRANSIT_AREA` — Stop is outside routing area  
- ⚠️ `INVALID_TIME_WINDOW` — Provided time window invalid
---

### 💡 Tips

- **Time formats**: `HH:mm` (24-hour).  
- **Units**: Distances in km/meters; times in seconds.
- **Priority**: `EARLIEST`/`LATEST` enforce hard windows; `AUTO` is flexible.  

