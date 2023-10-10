# API

## Authentication

### POST /api/user/otp-request

body:

```json
{
  "phone": "+976 8666 1488"
}
```

response: 200

### POST /api/user/otp-check

body:

```json
{
  "phone": "+976 8666 1488",
  "code": "6969"
}
```

response:

```json
{
  "token": "some bearer token string",
  "profile": {
    "name": "",
    "email": "",
    "phone": "+976 8666 1488"
  },
  "nats": {
    "servers": "nats.server.url:8443",
    "topic": "some-topic",
    "user": "user",
    "pass": "pass"
  }
}
```

**Note:** NATS is used as message broker, consider using [this package](https://pub.dev/packages/dart_nats)

## Chargers

### GET /api/devices/by-location

query:

```ts
{
  'bottom-left-longitude': number
  'bottom-left-latitude': number
  'upper-right-longitude': number
  'upper-right-latitude': number
}
```

response:

```ts
[
  {
    "_id": "64f85c8bfcb2a22c09a6fdc0",
    "deviceId": "XXX-XXX",
    "price": 8841,
    "waitingPrice": 1234,
    "location": [106.922542, 47.921084], //Longitude, Latitude
    "status": "Idle", // 'Idle' | 'Busy' | 'Repair' | 'Error'
  },
  ...
]
```

*example*: /api/devices/by-location?bottom-left-longitude=69.9&bottom-left-latitude=69.9&upper-right-longitude=127&upper-right-latitude=80

</br>

### Post /api/devices/:id/start

response: 200


### Post /api/devices/:id/finish

response 200


## Balance

### Post /api/customer/replenish

body:

```json
{
  "amount": 10069
}
```

responce:

```json
{
  "invoice_id": "e887c58b-7df2-4fe1-960c-6ab09b121488",
  "url": "https://s.qpay.mn/PHEc-oCO_O"
}
```
</br>

**Note:** after successful payment, the following message will be emitted from server:

```json
{
  "replenishment": {
    "status": "success",
    "amount": 10069
  }
}
```


### Post /api/customer/replenishment/:invoice_id

**Desc:** for manually checking the replenishment status

response:

```json
{
  "replenishment": {
    "status": "pending", // "pending" | "success" | "error" | "canceled"
    "amount": 10069
  }
}