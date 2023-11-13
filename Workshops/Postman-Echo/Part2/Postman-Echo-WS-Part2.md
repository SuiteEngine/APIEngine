# Postman Echo Workshop Part 2

## Introduction

In this second part, we will modify the payload from the previous part to pull data from the Customer record instead of using static values.

## Prerequisite

Before starting this workshop, ensure you have completed the first part, as we will be modifying the function and payload created in that session.

### Create a Copy of the BASICJSONPAYLOAD Function

1. Navigate to the API Function list and select the `BASICJSONPAYLOAD` API Function.
2. Access the function copy utility by going to Actions > Copy API Function.
3. When prompted, specify which information you would like to copy to the new function. Enter the following details:
   - Target API Set Code: `POSTMANECHO` — (Where the New API Function will be stored.)
   - Target Function Code: `BASICJSONPAYLOAD_BCRECORD` — (The name for the newly created function.)
   - Data To Copy:
     - General Function Information: `True` — (Copies the function's detailed information.)
     - API Variable: `True` — (Determines whether to copy the API Variables for the function.)
     - API Mapping: `True` — (Indicates if you want to copy the mappings; this will be covered in the next part.)
     - Credential: `True` — (Determines whether to copy the credential information.)

### Modify the Payload Variables

1. On the API Function List page, select the new function `BASICJSONPAYLOAD_BCRECORD` and navigate to the Payload page under API Variables > Payload.
2. Modify the payload object in Auto Build Sequence Line 10:
   - Before:
     - Value Processing Type: `Static`
     - Value Table No: (Blank)
   - After:
     - Value Processing Type: `Parameter Record`
     - Value Table No: `18`
3. Modify the name property in Auto Build Sequence Line 20:
   - Before:
     - Value Processing Type: `Static`
     - Static Value: `John Doe`
     - Value Table No: (Blank)
     - Value Field No: (Blank)
   - After:
     - Value Processing Type: `Parameter Record`
     - Static Value: (Blank)
     - Value Table No: `18`
     - Value Field No: `2`
4. Modify the phone property in Auto Build Sequence Line 30:
   - Before:
     - Value Processing Type: `Static`
     - Static Value: `123456789`
     - Value Table No: (Blank)
     - Value Field No: (Blank)
   - After:
     - Value Processing Type: `Parameter Record`
     - Static Value: (Blank)
     - Value Table No: `18`
     - Value Field No: `9`
5. Modify the email property in Auto Build Sequence Line 40:
   - Before:
     - Value Processing Type: `Static`
     - Static Value: `johndoe@example.com`
     - Value Table No: (Blank)
     - Value Field No: (Blank)
   - After:
     - Value Processing Type: `Parameter Record`
     - Static Value: (Blank)
     - Value Table No: `18`
     - Value Field No: `102`

### Prepare API Message Manually

Since we need to attach the customer record to the API Message, we have to create the API Message manually. We cannot simply execute the API function because the API engine would not know which customer to attach to the message.

1. From the API Function page, open the API Messages list page.
2. From the API Message list page, press the + new button to create a new message.
3. Ensure that the API Set Code and API Function code are correct:
   - API Set Code: `POSTMANECHO`
   - API Function Code: `BASICJSONPAYLOAD_BCRECORD`

### Add/Link Customer Record to the API Message

Once you have a new message, now we want to link the customer to the message.

1. Under the General section tab, find the parameters section.
2. Press the arrow down (v) beside the Parameters.
3. Click on the Add New Parameter Table Record.
4. You will be prompted with the objects list page.
5. Select the Customer Table (ID 18).
6. You will be prompted with the Customer list page.
7. Select Adatus Corporation and press "OK" at the bottom right.

### Execute the API Message

1. Go to Actions > Execute.
2. Scroll down to see the request generated and sent to the endpoint by the API Engine:
   {"name":"Adatum Corporation","phone":"","email":"robert.townes@contoso.com"}
