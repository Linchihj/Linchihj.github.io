# How can I embed the inline payment widget on to my website?

## Overview

You can add the inline payment widget to your website to help users complete payments for paid bookings made through the inline’s booking page using an HTTP request.

## Prerequisite

- Please submit your website domain name to inline.

> [!Note]
> The domain name submitted needs to be the same website used for embedding the inline payment widget.

## Procedure

1. Send an HTTP POST request to the inline payment intent API to create a payment intent.
For details, see *Step 1: Create the payment intent* section of this tutorial.
2. Configure the iframe for integrating the inline payment widget.
3. Integrate the inline payment widget into an iframe, and pass in the required parameters in the iframe source URL, and the Inline payment widget will handle all the payments and bookings for you. For details, see *Step 2: Configure the iframe* section of this tutorial.

>[!Note]
> inline recommends using a modal-like UI to render the iframe and close the modal after rendering.

4. Check the booking status or booking payment statuses via listening to the postMessages from the inline payment widget iframe

## Step 1: Create the payment intent

The payment intent created from inline payment intent API help you track a patron’s booking information and booking payment details. You can send an HTTPS POST request with a JSON request body to the inline payment intent API to create the payment intent.

```jsx
// For beta environment
POST https://sample.edu/#example

// For production environment
POST http://sample.com/?baseball=middle
```

The following example is an example request body:

```json
{
  "placeId": "string",
  "forkId": "string",
  "datetime": "string",
  "groupSize": 2,
  "numberOfKidChairs": 0,
  "numberOfKidSets": 0,
  "label": "string"
}
```

The JSON request body schema is as follows:

| Property | Data type | Required | Definition            |
| -------- | --------- | -------- | ----------------------|
| placeId  | string    | required | The company ID.       |
| forkId   | string    | required | The branch ID.        |
| datetime | string    | required | The reservation time. |
| groupSize| number    | required | The number of adults in a reservation. The minimum number of adults is 1. |
| numberOfKidChairs | number | optional | The number of children who don’t need highchairs in a reservation. By default, this value is 0.               |
| numberOfKidSets | number | optional | The number of children who need highchairs in a reservation. By default, this value is 0. |
| label    | string    | optional | The table tag.        |

The response body is the created payment intent, which returns the inline booking’s payment details in JSON format, and includes the payment widget URL. The following is a response body example:

```json
{
  "id": "string",
  "total": 0,
  "currency": "twd",
  "type": "prepay",
  "breakDown": {
    "pricePerPerson": 0,
    "minGroupSize": 0,
    "gracePeriod": 0,
    "pricePerTable": 0
  },
  "link": "string"
}
```

The JSON response body schema is as follows:

| Property     | Data type | Definition                                                 |
| ------------ | --------- | ---------------------------------------------------------- |
| id           | string    | The intent ID.                                             |
| total        | integer   | The total amount that will be charged for the booking. This includes no-show fees and any prepayments. The value can be 0 if no payment is required.|
| currency     | string    | The currency code in ISO 4217 format.                      |
| type         | string    | The booking payment type, which can be prepay or cardToken. **prepay**: A booking deposit paid by credit card. **cardToken**: An authorization hold placed on the patron’s credit card. The patron’s card isn’t charged at this point, but they will be charged for late cancellation or no-shows.|
| breakdown    | object   | The paid booking details based on the pricing categories.   |
| >pprice      | number   | The deposit per person.                                     |
| >minGroupSize| number   | The minimum number of people required for a booking.        |
| >gracePeriod | number   | The allowed cancellation period of a paid booking.          |
| >tprice      | number   | The price per table.                                        |
| widgetURL    | string   | The payment widget URL for completed inline reservations.   |


> [!NOTE]
> The following properties in the created payment intent should be displayed on your website for your user’s reference: `pricePerPerson`, `minGroupSize`, `gracePeriod`, and `pricePerTable`.

## Step 2: Configure the iframe for payment widget

You need to configure the iframe parameters to embed the inline payment widget onto your website. inline recommends using a modal-like UI to render the iframe and close the modal after rendering. The iframe requires the following specifications:

- iframe URL: The URL in the `widgetURL` property of the payment intent from *Step 1: Create the payment intent*.
- The following are the required fields of the query string for the URL:

| Field           | Definition                                                             |
| --------------- | -----------------------------------------------------------------------|
| placeId         | The company ID.                                                        |
| forkId          | The branch ID.                                                         |
| intentId        | The intent ID. For details, see Step 1: Create the payment intent.     |
| language        | The language of the payment widget user interface. For example, en, zh.|
| pre_filled_form | The reservation information.                                           |

Note that for the `pre_filled_form` field, you need to encode JSON string of the reservation information object in a **base64 encoded string**. The reservation information object properties are as follows:

```jsx
{
	  Items: {
		reservationTime: 1677653497000, // Timestamp in ms
		groupSize: 2,                   // The number of adult
		numberOfKidChairs: 1,           // The number of children who don't need highchair, optional
		numberOfKidSet: 1,              // The number of children who need highchair, optional
		label: "table-tag",          // The tag of the table customer selected for this reservation, optional
		placeId: "company-id",
		forkId: "branch-id"
	},
	diner: {
		language: "en",
		name: "John Smith"
		Details:{
			"familyName": "里",
      "givenName": "賢治",
      "phoneticFamilyName": "さと",
      "phoneticGivenName": "けんじ"
    },
		phone: "+886123456789",
		gender: "1",
		email: "johnsmith@email-address.com"
	},
	customerNote: "customer's note"
}
```

| Property            | Data type | Requirement | Definition                                |
| --------------------| --------- | ----------- | ----------------------------------------- |
| Items               | object    | required    | Verifies the information returned from the third-party platform matches the information generated from the payment intent request sent to inline. This is used mainly to verify the booking details are correct.                      |
| > reservationTime   | string    | required    | The date and time of a reservation in ISO 8601 format.                                                                                |
| > groupSize         | number    | required    | The number of adult in a reservation.     |
| > numberOfKidChairs | number    | optional    | The number of children who don’t need highchairs in a reservation.                                                                |
| > numberOfKidSet    | number    | optional    | The number of children who need highchairs in a reservation.                                                                           |
| > label             | string    | optional    | The table tags of a reservation.          |
| > placeId           | string    | required    | The company ID.                           |
| > forkId            | string    | required    | The branch ID.                            |
| diner               | object    | required    | The patron of this reservation.           |
| > language          | string    | optional    | The patron’s selected language preference.|
| > name              | string    | optional    | The patron’s full name.                   |
| > Details           | object    | optional    | The patron’s name separated into multiple fields.                                                                                     |
| >> familyName       | string    | optional    | The patron’s last name.                   |
| >> givenName        | string    | optional    | The patron’s first name.                  |
| >> phoneticFamilyName| string   | optional    | The family name written phonetically.     |
| >> phoneticGivenName | string   | optional    | The given name written phonetically.      |
| > SMS-phone          | string   | optional    | The patron’s phone number. The phone number prefix must include the country code.                                                       |
| > gender            | number    | optional    | The patron’s gender.The following values and their definition as follows: 0: Female /1: Male /2:Unspecified                                                |
| > email           | string    | optional      | The patron’s email address.               |
| customerNote      | string    | required      | The patron’s note for this reservation.   |

## Step 3: Check the booking statuses and booking payment statuses via listening to the postMessages from the inline Payment Widget iframe

To check the booking statuses and booking payment statuses through the inline payment widget, you can listen to the postMessages sent from the iframe. The inline payment widget  communicates with your website through postMessages. The postMessage is a JSON string and can be parsed into an object with the following properties:

| Property | Type   | Definition                                                         |
| -------- | ------ | ------------------------------------------------------------------ |
| event    | string | The postMessage event name.                                        |
| data     | object | The data sent in the postMessage event.  It is various for different event.                                                                                   |

The following are the different postMessages `Event` types:

| Event                      | Type       | Definition                              |
| ---------------------------| ---------- | --------------------------------------- |
| booking-created            | Successful | A reservation was created successfully. |
| booking-session-expired    | Error      | The reservation has expired.            |
| booking-session-mismatched | Error      | The reservation information returned from the third-party platform doesn’t match the requested quote from inline.                 |
| booking-create-failed      | Error      | Failed to create a reservation.         |
| payment-session-expired    | Error      | The payment session has expired.        |
| payment-failed             | Error      | The payment failed.                     |
| payment-3ds-failed         | Error      | The 3D Secure (3DS) verification of the payment failed.                                                                             |
| payment-form-height-update | Other      | The inline payment widget has been added to an iframe.                                                                             |

The payment widget sends a postMessage with a `payment-form-height-update` event to inform your website of the iframe height in pixel:

```jsx
{
    "event": "payment-form-height-update",
    "data": {
        "height": 896
    }
}
```

The `booking-created` event will be sent when a reservation is created successfully and the returned data is different for free and paid bookings.

1. The following is an example of a `booking-created` event when a **free** booking was successfully created:

```JSX
{
    "event": "booking-created",
    "data": {
        "requirePayment": false,
        "reservationId": "my-reservation-id",
    }
}
```

1. The following is an example of a `reservation-created` event when a **paid** booking was successfully created:

```JSX
{
    "event": "booking-created",
    "data": {
        "requirePayment": true,
        "reservationId": "my-reservation-id",
        "paymentMeta": {
            "paymentType": "VISA",
            "lastFour": "4242",
            "amount": 600,
            "currencyMultiplier": 1,
            "currency": "TWD"
        }
    }
}
```

The following are examples of error events returned for failed reservations:

`booking-session-expired`

```JSX
{
    event: 'booking-session-expired',
    data: {
        code: 903001,
        message: 'Payment intent not found or expired',
        reason: 'Payment intent not found or expired'
    }
}
```

`booking-session-expired`

```JSX
{
    event: 'booking-session-expired',
    data: {
        code: 903003,
        message: 'Payment intent already used',
        reason: 'Payment intent already used'
    }
}
```

`booking-session-mismatched`

```JSX
{
    event: 'booking-session-mismatched',
    data: {
        code: 903002,
        message: 'reservation session expired',
        reason: 'reservation session expired'
    }
}
```