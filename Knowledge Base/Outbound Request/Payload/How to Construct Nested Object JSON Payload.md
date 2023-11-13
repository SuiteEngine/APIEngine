# How to Construct JSON Payload

## Summary / Background

Often, we need to construct a JSON payload with a nested structure. In this article, we will demonstrate how to construct the following JSON structure:

```json
{
  "name": "John Doe",
  "address": {
    "line 1": "123 Main St",
    "line 2": "Unit 1",
    "City": "Toronto"
  }
}
```

## Constructing the Payload Structure using API Payload Variable

1. On the API Function List page, navigate to the Payload page under API Variables > Payload.
1. Create a new payload variable for defining the JSON object. Enter the following details:

   - Name: `payload`
   - Variable Value Type: `Object`
   - Value Processing Type: `Static`
   - API Name Processing Type: `Static`
   - Static Name: `payload`
   - Auto Build Request: `true`
   - Auto Build Sequence: `10`
   - Parent: (leave this field empty)

1. Add a new line for the name property:

   - Name: `name`
   - Variable Value Type: `Key Value Text`
   - Value Processing Type: `Static`
   - Static Value: `John Doe`
   - API Name Processing Type: `Static`
   - Static Name: `name`
   - Auto Build Request: `true`
   - Auto Build Sequence: `20`
   - Parent: `payload`

1. Add a new line for the address object:

   - Name: `address`
   - Variable Value Type: `Object`
   - Value Processing Type: `Static`
   - API Name Processing Type: `Static`
   - Static Name: `address`
   - Auto Build Request: `true`
   - Auto Build Sequence: `30`
   - Parent: `payload`

1. Add a new line for the address line 1 property:

   - Name: `line 1`
   - Variable Value Type: `Key Value Text`
   - Value Processing Type: `Static`
   - Static Value: `123 Main St`
   - API Name Processing Type: `Static`
   - Static Name: `line 1`
   - Auto Build Request: `true`
   - Auto Build Sequence: `40`
   - Parent: `address`

1. Add a new line for the address line 2 property:

   - Name: `line 2`
   - Variable Value Type: `Key Value Text`
   - Value Processing Type: `Static`
   - Static Value: `Unit 1`
   - API Name Processing Type: `Static`
   - Static Name: `line 2`
   - Auto Build Request: `true`
   - Auto Build Sequence: `50`
   - Parent: `address`

1. Add a new line for the address city property:

   - Name: `city`
   - Variable Value Type: `Key Value Text`
   - Value Processing Type: `Static`
   - Static Value: `Toronto`
   - API Name Processing Type: `Static`
   - Static Name: `city`
   - Auto Build Request: `true`
   - Auto Build Sequence: `60`
   - Parent: `address`

1. Return to the API function page and confirm the following setup:

   - HTTP Method: `Post`
   - Request Data Type: `JSON`

1. Execute the message and observe the request:

```json
{
  "name": "John Doe",
  "Address": { "Line 1": "Main St", "Line 2": "Unit 1", "City": "Toronto" }
}
```

This process demonstrates how to construct a nested object in a JSON payload. Remember to adjust the variable values and structures according to your specific endpoint requirements.
