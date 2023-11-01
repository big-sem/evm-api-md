# API

## Authentication

### POST /api/v1/user/otp-request

body:

```ts
{
  phone: "+976 8666 1488"
}
```

response: 200

### POST /api/v1/user/otp-check

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

**Note:** NATS is used as event bus, consider using [this package](https://pub.dev/packages/dart_nats)

## Chargers

### GET /api/v1/devices/by-location

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

*example*: /api/v1/devices/by-location?bottom-left-longitude=69.9&bottom-left-latitude=69.9&upper-right-longitude=127&upper-right-latitude=80

</br>

### Post /api/v1/devices/:id/start

response: 200


### Post /api/v1/devices/:id/finish

response: Purchase id

```ts
{ 
  id: "12d76c8bfca2a22c07a6fca1"
}
```

## Purchases

### Get /api/v1/payment-options

responce:

```ts
[
  {
    option: string,
    icon: string,
    title: string,
    disabled: true
  }
]
```

### Post /api/v1/purchases/:id/pay

params: `id` is purchase id obtained from the request above

body:

```ts
{ 
  option: string 
}
```

responce:

```ts
//EVM
{
  option: 'EVM',
  invoice_id: string, //purchase id
}
//QPAY
{
  option: 'QPAY',
  invoice_id: string,
  url: string
}
//Other payment options are disabled for now
```

**Note:** use url_launcher (or any other similar package) to open url for payment. After successful payment, the following message will be emitted from the server:

```ts
{
  payment: {
    status: 'success' | 'fail',
  }
}
```

### Post /api/v1/purchases/:id/status

### Post /api/v1/purchases/:id/feedback

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
*example*: /api/v1/purchases/12d76c8bfca2a22c07a6fca1/feedback

## Balance

### Post /api/v1/customer/replenish

body:

```ts
{
  amount: number,
  option: string
}
```

responce:

```ts
//QPAY
{
  option: 'QPAY'
  invoice_id: string,
  url: string
}
//Other payment options are disabled for now
```
</br>

**Note:** after successful replenishment, the following message will be emitted from the server:

```ts
{
  replenishment: {
    status: "success",
    amount: 10069
  }
}
```


### Post /api/v1/customer/replenishment/

**Desc:** for manually checking the replenishment status

body:

```ts
{
  option: string
  invoice_id: string
}
```

response:

```ts
{
  replenishment: {
    status: 'pending' | 'success' | 'error' | 'canceled'
    amount: number
  }
}
