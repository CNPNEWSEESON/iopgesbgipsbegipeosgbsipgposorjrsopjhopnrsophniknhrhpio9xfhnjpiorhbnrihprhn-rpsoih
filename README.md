# Stats API (GET)

Base URL: `https://wl.cnpxdev.com`

API ชุดนี้ใช้สำหรับดูสถิติการรันสคริปต์ และสามารถเรียกใช้งานได้เฉพาะ Admin เท่านั้น

## Authentication

ทุก Request ต้องส่ง `x-admin-token` ใน Header โดยค่าที่ส่งต้องตรงกับ `ADMIN_TOKEN` ที่ตั้งไว้บน Server

```http
x-admin-token: <ADMIN_TOKEN>
```

API จะส่งข้อมูลกลับมาเป็น JSON โดยตรง

ไม่สามารถเปิด Endpoint ผ่าน URL บน Browser โดยตรงได้ เนื่องจาก Browser ไม่สามารถกำหนด `x-admin-token` ในการเปิด URL แบบปกติได้ แนะนำให้ใช้ `curl`, Postman หรือเขียนโค้ดเรียก API แทน

หากไม่ส่ง Token หรือ Token ไม่ถูกต้อง Server จะตอบกลับด้วย HTTP `403`

```json
{
  "Message": "Forbidden"
}
```

## ตัวอย่างการเรียก API

### curl

```bash
curl -H "x-admin-token: <ADMIN_TOKEN>" https://wl.cnpxdev.com/api/stats/overview
```

### PowerShell

```powershell
Invoke-RestMethod `
  -Uri "https://wl.cnpxdev.com/api/stats/overview" `
  -Headers @{ "x-admin-token" = "<ADMIN_TOKEN>" }
```

### JavaScript

```js
fetch("https://wl.cnpxdev.com/api/stats/overview", {
  headers: {
    "x-admin-token": "<ADMIN_TOKEN>"
  }
})
  .then(res => res.json())
  .then(console.log);
```

หากต้องการเรียก API ตัวอื่น ให้เปลี่ยนเฉพาะส่วนท้ายของ URL

---

## GET `/api/stats/overview`

ใช้ดูจำนวนสถิติทั้งหมด

### Response

```json
{
  "totalRuns": 120,
  "totalApiCalls": 340,
  "uniqueUsers": 45,
  "uniqueMaps": 8
}
```

| ฟิลด์           | รายละเอียด                                                              |
| --------------- | ----------------------------------------------------------------------- |
| `totalRuns`     | จำนวนครั้งที่มีการรันสคริปต์ทั้งหมดจาก `/api/execute`                   |
| `totalApiCalls` | จำนวนการเรียก API ทั้งหมด โดยรวม `getInfo` ที่ผ่านและจำนวนการรันสคริปต์ |
| `uniqueUsers`   | จำนวน Username ที่ไม่ซ้ำกัน                                             |
| `uniqueMaps`    | จำนวน Map ที่ไม่ซ้ำกัน                                                  |

---

## GET `/api/stats/maps`

แสดง Map ที่มีการรันมากที่สุด 20 อันดับ โดยเรียงจากจำนวนรันมากไปน้อย

### Response

```json
[
  {
    "mapname": "Blox Fruits",
    "placeId": 2753915549,
    "runs": 80,
    "users": 30
  }
]
```

| ฟิลด์     | รายละเอียด                                                       |
| --------- | ---------------------------------------------------------------- |
| `mapname` | ชื่อ Map                                                         |
| `placeId` | Place ID ล่าสุดที่ได้รับของ Map นั้น หากไม่มีข้อมูลจะเป็น `null` |
| `runs`    | จำนวนครั้งที่มีการรันใน Map                                      |
| `users`   | จำนวนผู้ใช้ที่ไม่ซ้ำกันใน Map                                    |

---

## GET `/api/stats/users`

แสดงผู้ใช้ที่มีจำนวนการรันมากที่สุด 20 อันดับ

### Response

```json
[
  {
    "username": "Player1",
    "runs": 15,
    "last_mapname": "Blox Fruits",
    "last_run_at": "2026-10-06T10:00:00.000Z"
  }
]
```

| ฟิลด์          | รายละเอียด                              |
| -------------- | --------------------------------------- |
| `username`     | Username ของผู้ใช้                      |
| `runs`         | จำนวนครั้งที่ผู้ใช้รันสคริปต์           |
| `last_mapname` | Map ล่าสุดที่ผู้ใช้รัน                  |
| `last_run_at`  | เวลาที่รันล่าสุดในรูปแบบ ISO 8601 (UTC) |

---

## GET `/api/stats/recent`

แสดงรายการรันล่าสุด 50 ครั้ง โดยรายการล่าสุดจะอยู่ด้านบน

Server จะเก็บข้อมูลการรันย้อนหลังสูงสุด 100 ครั้ง

### Response

```json
[
  {
    "username": "Player1",
    "mapname": "Blox Fruits",
    "placeId": 2753915549,
    "at": "2026-10-06T10:00:00.000Z"
  }
]
```

| ฟิลด์      | รายละเอียด                                   |
| ---------- | -------------------------------------------- |
| `username` | Username ของผู้ใช้                           |
| `mapname`  | Map ที่รัน                                   |
| `placeId`  | Place ID ของ Map หากไม่มีข้อมูลจะเป็น `null` |
| `at`       | เวลาที่รันในรูปแบบ ISO 8601 (UTC)            |

---

## Error

| Status | Response                  | สาเหตุ                                                                              |
| ------ | ------------------------- | ----------------------------------------------------------------------------------- |
| `403`  | `{"Message":"Forbidden"}` | ไม่ได้ส่ง `x-admin-token`, Token ไม่ถูกต้อง หรือ Server ไม่ได้ตั้งค่า `ADMIN_TOKEN` |

---

## การเก็บข้อมูล

ข้อมูลสถิติจะถูกเก็บไว้ในไฟล์ `stats.json`

Server จะเขียนข้อมูลลงไฟล์ทุก ๆ 5 วินาที ดังนั้นหาก Server หยุดทำงานหรือเกิด Crash ก่อนถึงรอบการบันทึก ข้อมูลที่เกิดขึ้นในช่วงนั้นอาจไม่ได้ถูกบันทึกลงไฟล์

หากยังไม่มีข้อมูลในส่วน `maps`, `users` หรือ `recent` API จะส่ง Array ว่างกลับมา

```json
[]
```

ส่วน `/api/stats/overview` จะคืนค่าเป็น `0` ในทุกฟิลด์เมื่อยังไม่มีข้อมูล
