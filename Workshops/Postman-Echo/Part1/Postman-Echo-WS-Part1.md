# Postman Echo Workshop Part 1

## Introduction

In the first part of this workshop, we will focus on constructing a JSON payload with static values. The payload that we are going to construct is in this format:

```json
{
  "name": "John Doe",
  "phone": "123456789",
  "email": "johndoe@example.com"
}
```

### Create New API Set and Function

1. Create a new API Set called `POSTMANECHO`
1. Create a new API Credential for the `POSTMANECHO` with the following details

   - API Set Code : `POSTMANECHO`
   - API Function Code : (leave this field blank)
   - API Endpoint URL : `https://postman-echo.com`
   - Autorization Type : `No Autorization`

1. Create a new API Function with the following details

   - Fuction Code : `BASICJSONPAYLOAD`
   - Description : `Demonstrate how to Construct Basic JSON payload`
   - HTTP Method : `POST`
   - Static Path : `post`
   - Request Data Type : `JSON`

### Create Payload Variables

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
