# EP13.2 Part 2/2 — InfluxDB to Node-RED Dashboard

## Goal

ดึงข้อมูลย้อนหลังจาก InfluxDB แล้วแสดง Trend บน Node-RED Dashboard

```text
InfluxDB → Node-RED Query → Dashboard Chart
```

---

## Flux Query: Production Count Trend

```flux
from(bucket: "plc_data")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "plc_status")
  |> filter(fn: (r) => r._field == "count")
  |> aggregateWindow(every: 10s, fn: last, createEmpty: false)
  |> yield(name: "count")
```

---

## Function Node: Format Chart Data

ใช้หลัง InfluxDB query เพื่อแปลงข้อมูลให้อยู่ในรูปแบบที่ Chart ใช้งานได้

```js
let rows = msg.payload || [];

msg.payload = rows.map(r => {
  return {
    x: new Date(r._time).getTime(),
    y: Number(r._value)
  };
});

return msg;
```

---

## Suggested Dashboard Layout

| Group | Widget | Purpose |
|---|---|---|
| Current Status | Text / LED / Gauge | แสดงค่าปัจจุบันจาก MQTT |
| Historical Trend | Chart | แสดง Count ย้อนหลัง |
| Machine State Trend | Chart | แสดง Run / Alarm / Ready ย้อนหลัง |

---

## Teaching Point

Node-RED Dashboard เหมาะมากสำหรับการสอน เพราะเห็นทั้ง Flow, Data Pipeline และ Dashboard ในเครื่องมือเดียว

Grafana เหมาะสำหรับตอนถัดไป เมื่อข้อมูลถูกจัดเก็บใน InfluxDB เรียบร้อยแล้ว
