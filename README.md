# API

## Authentication

### POST /api/user/otp-request

body:

```ts
{
  phone: "+976 8666 1488"
}
```

response: 200

### POST /api/user/otp-check

body:

```ts
{
  phone: "+976 8666 1488",
  code: "6969"
}
```

response:

```ts
{
  token: "some bearer token string",
  profile: {
    name: "",
    email: "",
    phone: "+976 8666 1488",
    balance: 0,
  },
  nats: {
    servers: "nats.server.url:8443",
    topic: "some-topic",
    user: "user",
    pass: "pass"
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
    _id: ObjectId,
    deviceNumber: string,
    price: number,
    waitingPrice: number,
    location: [number, number], //Longitude, Latitude
    state: 'Idle' | 'Busy' | 'Repair' | 'Error'
  },
  ...
]
```

*example*: /api/devices/by-location?bottom-left-longitude=69.9&bottom-left-latitude=69.9&upper-right-longitude=127&upper-right-latitude=80

</br>

### Post /api/devices/:id/start

response: 200


### Post /api/devices/:id/finish

response: Purchase id

```ts
{ 
  id: "12d76c8bfca2a22c07a6fca1"
}
```

## Purchases

### Post /api/purchases/:id/feedback

params: `id` is purchase id obtained from request above

body:

```ts
{
  feedback: {
    score: number, // 1-5
    description: string
  }
}
```

response: 200
*example*: /api/purchases/12d76c8bfca2a22c07a6fca1/feedback

## Balance

### Post /api/customer/replenish

body:

```ts
{
  amount: 10069
}
```

responce:

```ts
{
  invoice_id: "e887c58b-7df2-4fe1-960c-6ab09b121488",
  url: "https://s.qpay.mn/PHEc-oCO_O"
}
```
</br>

**Note:** after successful payment, the following message will be emitted from server:

```ts
{
  replenishment: {
    status: "success",
    amount: 10069
  }
}
```


### Post /api/customer/replenishment/:invoice_id

**Desc:** for manually checking the replenishment status

response:

```ts
{
  replenishment: {
    status: 'pending' | 'success' | 'error' | 'canceled'
    amount: number
  }
}