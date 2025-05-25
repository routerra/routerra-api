# routerra-api
Repository for Routerra API documentation and client side code

About us: https://routerra.io
Email: info@routerra.io

# Route Optimization API Reference

**Base URL**  
```
https://api.routerra.io/external/v1
```

**Authentication**  
Include your API key in every request header:  
```
API-KEY: <your-key>
```

---

## POST /optimize

Optimize a multi-stop route.

### Request

```http
POST /external/v1/optimize HTTP/1.1
Host: api.routerra.io
API-KEY: your-key
Content-Type: application/json

{
  "startLocation": Location,
  "startTime":   "HH:mm",
  "stops":       Stop[],
  "finishLocation": Location | null,
  "optimizeSettings": OptimizationSettings
}
```

| Field              | Type                            | Req’d | Description                      |
|--------------------|---------------------------------|:-----:|----------------------------------|
| `startLocation`    | `Location`                      |  ✔️   | Where the route begins           |
| `startTime`        | `string` (`HH:mm`)              |  ✔️   | Local departure time             |
| `stops`            | `Stop[]`                        |  ✔️   | List of intermediate stops       |
| `finishLocation`   | `Location` \| `null`            |  ❌   | Optional final point             |
| `optimizeSettings` | `OptimizationSettings`          |  ✔️   | Global flags & vehicle info      |

#### Location

```json
{
  "latitude":  number,
  "longitude": number,
  "address":   string | null
}
```

#### Stop

```json
{
  "location":         Location,
  "arrivalRangeFrom": "HH:mm" | null,
  "arrivalRangeTo":   "HH:mm" | null,
  "serviceTime":      number | null,
  "priority":         "AUTO" | "EARLIEST" | "LATEST",
  "stopSide":         "ANY" | "LEFT" | "RIGHT"
}
```

#### OptimizationSettings

```json
{
  "avoidTolls":      boolean,
  "liveRoadData":    boolean,
  "avoidHighway":    boolean,
  "vehicleType":     "BIKE" | "SCOOTER" | "CAR" | "VAN" | "TRUCK",
  "optimizeBy":      "distance" | "time",
  "serviceTime":     number,
  "stopSide":        "ANY" | "LEFT" | "RIGHT"
}
```

---

### Sample cURL

```bash
curl -X POST "https://api.routerra.io/external/v1/optimize"   -H "API-KEY: your-key"   -H "Content-Type: application/json"   -d '{
        "startLocation": { "latitude":52.2297, "longitude":21.0122, "address":"Warsaw" },
        "startTime": "08:00",
        "stops": [ /* … */ ],
        "finishLocation": null,
        "optimizeSettings": { /* … */ }
      }'
```

---

### Response

```json
{
  "routeErrorType":       string | null,
  "statistics":           RouteStatistics,
  "startLocation":        Location,
  "stops":                OptimizedStop[],
  "finishLocation":       Location | null,
  "finishDriveTime":      number | null,
  "finishDriveDistance":  number | null,
  "finishLocationArrival": "HH:mm"
}
```

#### RouteStatistics

```json
{ "distance": number, "time": number, "stops": number }
```

#### OptimizedStop

```json
{
  "position":        number,
  "location":        Location,
  "waitTime":        number | null,
  "driveTime":       number,
  "driveDistance":   number,
  "stopErrorType":   string | null,
  "expectedArrival": "HH:mm"
}
```

---

### Error Handling

Error responses include:

```json
{ "error": { "code": string, "message": string } }
```

| HTTP  | When                                     |
|-------|------------------------------------------|
| 400   | Bad JSON / missing fields                |
| 401   | Missing or invalid API-KEY               |
| 500   | Server error — retry or contact support  |

---

### Error Types

**stopErrorType**  
- `CANT_VISIT_TIME_WINDOW` — Time window constraints can’t be met  
- `OUTSIDE_TRANSIT_AREA` — Stop is outside routing area  
- `INVALID_TIME_WINDOW` — Provided time window invalid  

**routeErrorType**  
- `MATRIX_CALCULATION_FAILED` — Can’t compute distance/time matrix  
- `ROUTE_OPTIMIZATION_FAILED` — Algorithm failure  
- `TOO_MANY_LOCATIONS_FOR_UNKNOWN_REGION` — Too many stops in undefined region  

---

### Tips

- Times = `HH:mm` (24h)  
- Distances = km/meters; times = seconds  
- Per-stop `serviceTime` falls back to global  
- Use `EARLIEST`/`LATEST` for hard windows  
