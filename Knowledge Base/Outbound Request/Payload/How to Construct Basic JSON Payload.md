# How to Construct JSON Payload

## Summary / Background

Interacting with external RESTful web services often requires sending data in JSON format. The API Engine facilitates the creation of such payloads. This article guides you through constructing various payload structures, starting with a basic example. We'll demonstrate how to send data to an endpoint containing a name, phone number, and email address.

Here's the payload we are going to construct:

```json
{
  "name": "John Doe",
  "phone": "123456789",
  "email": "johndoe@example.com"
}
```

## Constructing the Payload Structure using API Payload Variable

1. On the API Function List page, navigate to the Payload page under API Variables > Payload.
1. Create a new payload variable for defining the JSON object. Enter the following details:

   - Name : `payload`
   - Variable Value Type : `Object`
   - Value Processing Type : `Static`
   - API Name Processing Type : `Static`
   - Static Name :`payload`
   - Auto Build Request : `true`
   - Auto Build Sequence : `10`
   - Parent : ``

1. Add a new line for the name property:

   - Name : `name`
   - Variable Value Type : `Key Value Text`
   - Value Processing Type : `Static`
   - Static Value : `John Doe`
   - API Name Processing Type : `Static`
   - Static Name :`name`
   - Auto Build Request : `true`
   - Auto Build Sequence : `20`
   - Parent : `payload`

1. Add a new line for the phone property:

   - Name : `phone`
   - Variable Value Type : `Key Value Text`
   - Value Processing Type : `Static`
   - Static Value : `1234567890`
   - API Name Processing Type : `Static`
   - Static Name :`phone`
   - Auto Build Request : `true`
   - Auto Build Sequence : `30`
   - Parent : `payload`

1. Add a new line for the email property:

   - Name : `email`
   - Variable Value Type : `Key Value Text`
   - Value Processing Type : `Static`
   - Static Value : `johndoe@example.com`
   - API Name Processing Type : `Static`
   - Static Name :`email`
   - Auto Build Request : `true`
   - Auto Build Sequence : `40`
   - Parent : `payload`

1. Return to the API function page and confirm the following setup:

   - HTTP Method : `Post`
   - Request Data Type : `JSON`

1. Execute the message and observe the request:

```json
{ "name": "John Doe", "phone": "123456789", "email": "johndoe@example.com" }
```

This process demonstrates the basic setup for constructing a JSON payload. Remember to adjust the variable values and structures according to your specific endpoint requirements.
