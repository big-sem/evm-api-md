# API

## Authentication

### POST /api/v1/user/otp-request

body:

```ts
{
  phone: "+976 8666 1488" //international format
}
```

response: 200

### POST /api/v1/user/otp-check

body:

```ts
{
  phone: string
  code: string //0000 - for dev
}
```

response:

```ts
{
  token: string
  profile: {
    name: string
    email: string
    phone: string
    balance: number
  },
  device?: {
    _id: string
    deviceNumber: string
    location: [number, number] //array of lenght 2
    waitingPrice: number
    price: number
  }
  nats: {
    servers: string
    topic: string
    user: string
    pass: string
  }
}
```

### POST /api/v1/user/me

response:

```ts
{
  token: string
  profile: {
    name: string
    email: string
    phone: string
    balance: number
  },
  device?: {
    _id: string
    deviceNumber: string
    location: [number, number] //array of lenght 2
    waitingPrice: number
    price: number
  }
  nats: {
    servers: string
    topic: string
    user: string
    pass: string
  }
}
```

**Note:** NATS is used as event bus, consider using [this package](https://pub.dev/packages/dart_nats)

### POST /api/v1/customer/update-notification-token

body:

```ts
{
  FCMToken: string
}
```

response: 200

### POST /api/v1/customer/update-info

body

```ts
{
    name: string
    email: string
}
```

response: 200


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
    _id: string
    deviceNumber: string
    price: number
    waitingPrice: number
    location: [number, number] //Longitude, Latitude
    state: 'Idle' | 'Busy' | 'Repair' | 'Error'
  },
  ...
]
```

*example*: /api/v1/devices/by-location?bottom-left-longitude=69.9&bottom-left-latitude=69.9&upper-right-longitude=127&upper-right-latitude=80

</br>

### Post /api/v1/devices/:id/start

response: 200

This message will be emitted from the server every 2-5 seconds

```ts
//{subject}.CHARGING
{
  energy: number
  waitingTime: number
  amount: number
}
```

If server

### Post /api/v1/devices/:id/finish

response: Purchase id

```ts
{ 
  id: "12d76c8bfca2a22c07a6fca1"
}
```

## Purchases

### Get /api/v1/payment-options

response:

```ts
[
  {
    option: string
    icon: string
    title: string
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

response:

```ts
//EVM
{
  option: 'EVM'
  invoiceId: string //purchase id
  newBalance: number
}
//QPAY
{
  option: 'QPAY'
  invoiceId: string
  url: string
}
//Other payment options are disabled for now
```

**Note:** use url_launcher (or any other similar package) to open url for payment. After successful payment, the following message will be emitted from the server:

```ts
//{subject}.PAYMENT
{
  status: 'success' | 'fail'
  amount: number
}
```

### Post /api/v1/purchases/:id/status

**Desc:** for manually checking the payment status

body:

```ts
{
  invoiceId: string
}
```

response:

```ts
{
  status: 'pending' | 'success' | 'error' | 'canceled'
  amount: number
}
```

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

### Get /api/v1/purchases

params:

```ts
{
  page: number // from 1
  'page-size': number //optional, 50 by default
}
```

response: **check postman**

## Balance

### Post /api/v1/customer/replenish

body:

```ts
{
  amount: number
  option: string
}
```

response:

```ts
//QPAY
{
  option: 'QPAY'
  invoiceId: string
  url: string
}
//Other payment options are disabled for now
```
</br>

**Note:** after successful replenishment, the following message will be emitted from the server:

```ts
//{subject}.REPLENISHMENT
{
  status: "success"
  amount: 10069
}
```


### Post /api/v1/customer/replenishment/

**Desc:** for manually checking the replenishment status

body:

```ts
{
  invoiceId: string
}
```

response:

```ts
{
  status: 'pending' | 'success' | 'error' | 'canceled'
  amount: number
}
