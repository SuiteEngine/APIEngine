# Joke API Workshop with API Engine - Part 4

## Introduction

Welcome to Part Four of our six-part JOKE API Workshop series. In Part Three, we made significant progress by moving from static URL construction to dynamic construction using runtime variables. We learned how to use **Parameter Record** variables, which allow us to pull values from Business Central table records when making API calls.

However, we still had a limitation: the number of jokes returned was hardcoded to 10. In real-world scenarios, you often want to give users the ability to specify simple values—like a quantity, a date range, or a filter—without requiring you to create table records for every possible value.

This is where **Key-Value Parameters** become essential. Think of them as the equivalent of prompting a user for input in a coding solution. Instead of storing values in tables, you can pass simple key-value pairs directly to the API function, making your integration much more flexible and user-friendly.

In this session, we'll build upon the concepts from Part Three and introduce a second type of runtime parameter that works alongside Parameter Record variables, giving you complete control over your API calls.

### Objectives

- Continue constructing API variables (specifically URL parameters) with values provided at runtime.
- Understand and implement **Key-Value parameters** through the user interface.
- Learn how to combine multiple parameter types in a single API call (both Table Record and Key-Value).
- Explore additional stages of the API message lifecycle, particularly the **Load Response** step.
- View and analyze the Data Buffer to understand how API responses are structured.

## Requirements

Before participating in this workshop, you should have:

- Completed Parts One, Two, and Three of the series.
- Access to a SaaS Business Central environment with the Suite Engine API Engine Extension installed.
- Created the API Set, Credentials, Functions, and Variables from the previous workshops.
- Understanding of runtime variables and Parameter Record processing from Part Three.
- At least one Item Category record configured with a valid Joke API category (e.g., "Miscellaneous" or "Programming").

For additional reference, the detailed documentation for the Joke API endpoint is available at [https://jokeapi.dev/](https://jokeapi.dev/). You may also find our YouTube tutorial series helpful: [Joke API Video Tutorials](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ).

## Review: What We've Accomplished So Far

Let's take a moment to review our progress through this workshop series:

### Part 1: The Foundation
- Created our first API Set (JOKEAPI)
- Configured API Credentials with the base endpoint URL
- Built basic API Functions with static configurations
- Executed our first API calls and viewed responses

### Part 2: URL Construction
- Learned about URL structure (Base URL + Path + Parameters)
- Moved the base URL to the Credential record
- Configured Static Path and URL Path variables
- Added URL Parameters for query strings
- Copied API Functions for efficiency

### Part 3: Runtime Variables with Table Records
- Introduced the concept of runtime variables
- Learned about the API Message Lifecycle
- Created Parameter Record variables
- Manually created API Messages and attached parameters
- Used the Item Category table to provide dynamic category values

### Current State: Still One Static Value

At this point, we have an API function called **GETJOKESBYCATEGORY** (or similar) that:

✅ Dynamically retrieves the joke **category** from a Parameter Record (Item Category table)  
❌ Still uses a **static value of 10** for the number of jokes to retrieve

While this is much better than having everything hardcoded, we still can't let the user specify how many jokes they want without modifying the function configuration.

## Understanding Parameter Types

Before we proceed, let's clarify the two types of runtime parameters we can use in the API Engine:

### 1. Parameter Record (Covered in Part 3)

- **Purpose:** Pull a value from a specific field in a Business Central table record
- **Use Case:** When you need data that already exists in BC tables (customers, items, categories, etc.)
- **Example:** Get the category name from an Item Category record
- **Advantages:**
  - Leverages existing BC data
  - Maintains data consistency
  - Good for standardized values
- **Limitations:**
  - Requires records to exist in a table
  - Can be overkill for simple values

### 2. Key-Value Parameter (Today's Focus)

- **Purpose:** Pass a simple value directly without needing a table record
- **Use Case:** When you need to specify simple inputs like numbers, text, dates, or boolean flags
- **Example:** Specify the number of jokes to retrieve (e.g., `amount=5`)
- **Advantages:**
  - Simple and flexible
  - No need to create table records
  - Perfect for user-specified values
- **Limitations:**
  - Values are not stored permanently (unless you code that separately)
  - No built-in validation against BC table data

### When to Use Each Type

| Scenario | Recommended Parameter Type |
|----------|---------------------------|
| User selects a customer from a list | Parameter Record (Customer table) |
| User enters a quantity | Key-Value Parameter |
| User selects an item category | Parameter Record (Item Category table) |
| User enters a date range | Key-Value Parameter |
| User specifies a filter value | Key-Value Parameter |
| User selects a location | Parameter Record (Location table) |

In many real-world scenarios, you'll use **both types together** in the same API call, which is exactly what we'll demonstrate today.

## Modifying the URL Parameter to Use Key-Value

Now let's modify our existing function to make the `amount` parameter dynamic instead of static.

### Step 1: Navigate to the API Function

1. Search for **API Functions** in Business Central.
2. Filter to the **JOKEAPI** API Set.
3. Open the **GETJOKESBYCATEGORY** function (or whatever you named your function from Part 3).

### Step 2: Open the URL Parameters

1. On the API Function card, navigate to **API Variables** > **URL Parameters**.
2. Locate the **amount** parameter (the one currently set to a static value of `10`).

### Step 3: Change the Value Processing Type

1. Edit the **amount** URL parameter record.
2. Change the following fields:
   - **Value Processing Type:** Change from `Static` to `Parameter Value`
   - **Variable Value Type:** Should remain `Key Value Text` (or similar)
   - **Static Name:** Ensure this is set to `amount` (lowercase)
   - **Var Value Null Behavior:** Set to `Send Empty Value` (or similar option)

**Why lowercase for Static Name?**

The **Static Name** represents the actual key that will be sent in the URL query string to the external API. Many APIs, including the Joke API, are case-sensitive and expect lowercase parameter names. Even though Business Central often uses uppercase for internal field names, we need to respect the external API's requirements.

**What is Var Value Null Behavior?**

This setting determines what happens if no parameter value is provided at runtime:
- **Send Empty Value:** The parameter will be included in the URL but with no value (e.g., `?amount=`)
- **Skip Variable:** The parameter will be omitted from the URL entirely
- **Use Default:** A fallback value will be used

For this example, we'll allow it to send an empty value. The Joke API will use its own default if no amount is specified.

### Step 4: Save the Configuration

Close the URL Parameters page and return to the API Function card. Your function is now configured to accept a runtime value for the amount parameter.

## Understanding the Updated Configuration

Let's review what we now have configured in our **GETJOKESBYCATEGORY** function:

### URL Path Variable: Category
- **Variable Scope:** URL Path
- **Name:** `Category`
- **Value Processing Type:** `Parameter Record`
- **Parameter Table No.:** `5722` (Item Category)
- **Parameter Field No.:** `3` (Description)

This means: "Look for an Item Category record in the parameters and use its Description field value."

### URL Parameter: Safe-Mode
- **Variable Scope:** URL Parameter
- **Name:** `Safe-Mode`
- **Variable Value Type:** `Text Value`
- **Value Processing Type:** `Static`
- **Static Value:** `safe-mode`

This means: "Always include `safe-mode` in the URL parameters as a tag."

### URL Parameter: Amount
- **Variable Scope:** URL Parameter
- **Name:** `amount`
- **Variable Value Type:** `Key Value Text`
- **Value Processing Type:** `Parameter Value` (changed from Static)
- **Static Name:** `amount` (the key name for the API)
- **Var Value Null Behavior:** `Send Empty Value`

This means: "Look for a parameter with the key `amount` and use its value in the URL query string."

## Creating a Message with Multiple Parameters

Now that our function is configured to accept both types of parameters, let's create an API Message and provide both values.

### Step 1: Navigate to API Messages

Instead of using the **Execute** button (which runs immediately without parameters), we need to manually create a message:

1. From the **API Functions** list, select your **GETJOKESBYCATEGORY** function.
2. Navigate to **Related** > **API Messages**.

Alternatively, you can search for **API Messages** directly and create a new one, then manually select the function.

### Step 2: Create a New API Message

1. On the **API Messages** page, click **+ New** to create a new message.
2. Ensure the **API Function Code** field is populated with your function (it should auto-fill if you navigated from the function).
3. Verify the **API Set Code** is set to `JOKEAPI`.
4. Check that the **Message Status** shows **Initialized**.

At this point, the message exists, but we haven't provided any parameters yet.

### Step 3: Add the First Parameter (Table Record)

We need to provide the category parameter first, which comes from a table record.

1. On the API Message card, scroll down to the **Parameters** section (or navigate to **Related** > **API Parameters**).
2. Click **Add New Parameter** and select **Table Record**.
3. You'll be prompted to select a table. Enter or search for table `5722` (Item Category).
4. A list of Item Category records will appear. Select the record with Description **Miscellaneous** (or any other valid Joke API category like "Programming").
5. The parameter is now added to the message.

You should see one parameter in the Parameters list, showing:
- **Value Type:** Table Record
- **Table Record Value:** A reference to the Item Category record you selected

### Step 4: Add the Second Parameter (Key-Value)

Now we'll add a simple key-value parameter to specify how many jokes we want.

1. Still in the Parameters section, click **Add New Parameter** and select **Key Value**.
2. A new parameter record will be created. Edit it with the following values:
   - **Key:** `amount` (lowercase)
   - **Value Type:** `Text` (or `Integer` if available)
   - **Value:** `5`

**Important:** The **Key** field must match the parameter name that the API Function is looking for. In our case, the function's URL Parameter variable is looking for a parameter with the key `amount`, so we use exactly that.

You should now see two parameters in the Parameters list:
1. A Table Record parameter (representing the Item Category)
2. A Key-Value parameter with Key=`amount` and Value=`5`

## Reviewing the API Message Lifecycle (Revisited)

Before we send the message, let's review the full API message lifecycle, including stages we haven't explored yet:

### Stage 1: Initialized ✅ (Current Status)
- **Description:** Message created, API Function selected
- **User Actions Available:**
  - Add or modify parameters
  - Edit request settings
  - Specify headers (advanced)
- **Next Step:** Send the request

### Stage 2: Request
- **Description:** Request built and sent to the API endpoint
- **What Happens:**
  - API Engine constructs the full URL using base URL, paths, and parameters
  - Variables are resolved using the provided parameter values
  - HTTP headers are prepared
  - Request payload is built (if applicable)
  - HTTP request is sent to the external API
  - System waits for response
- **Next Step:** Receive response

### Stage 3: Response ✅ (We'll reach this today)
- **Description:** Response received from the external API
- **What Happens:**
  - HTTP response status code is captured
  - Response headers are saved (if configured)
  - Response body is stored in the Response Message field
  - Message status changes to "Response"
- **User Actions Available:**
  - View the raw response
  - Review request details
  - Load response into Data Buffer
- **Next Step:** Load response into Data Buffer

### Stage 4: DataBuffer Loaded ✅ (We'll reach this today)
- **Description:** Response parsed and loaded into structured data buffer
- **What Happens:**
  - API Engine parses the response (JSON, XML, etc.)
  - Data is broken into structured records in the Data Buffer table
  - Hierarchical relationships are preserved
  - Arrays and objects are indexed
- **User Actions Available:**
  - View data buffer records
  - Process data buffer using mappings
- **Next Step:** Process data according to mappings

### Stage 5: Complete
- **Description:** Data has been processed and loaded into target tables
- **What Happens:**
  - API Mappings define how to move data from buffer to BC tables
  - Records are created, updated, or validated
  - Processing logs are generated
- **Note:** We'll cover this in the next video

## Sending the API Message

Now that we have both parameters configured, let's send the message and step through the lifecycle manually.

### Step 1: Send the Request

1. On the API Message card, click the **Send** action (in the Manual Processing group).
2. The API Engine will:
   - Construct the URL using both parameter values
   - Send the HTTP request to the Joke API
   - Receive the response
   - Update the message status

### Step 2: Verify the Response Status

After the send operation completes:

1. Check the **Message Status** field—it should now show **Response**.
2. Check the **HTTP Response Status Code**—it should show **200** (success).
3. Note the **Request Completion Timestamp**—this shows when the API call completed.

If you see an error code (like 400, 404, or 500), review the Query Request and Response sections for details about what went wrong.

### Step 3: Review the Query Request

Let's verify that both parameters were correctly used in the URL construction:

1. On the API Message card, locate the **Query Request** section.
2. You should see a URL similar to:
   ```
   https://v2.jokeapi.dev/joke/Miscellaneous?safe-mode&amount=5
   ```

**Breaking down this URL:**
- **Base URL:** `https://v2.jokeapi.dev` (from the API Credential)
- **Static Path:** `joke` (from the function's API Path field)
- **Variable Path:** `Miscellaneous` (from the Item Category Description field via Parameter Record)
- **Query Parameters:**
  - `safe-mode` (static URL parameter)
  - `amount=5` (from the Key-Value parameter)

Both parameters were successfully resolved and included in the final URL!

### Step 4: Review the Raw Response

1. On the API Message card, scroll to the **Response** section.
2. You'll see the raw JSON response from the Joke API.
3. The response should indicate that 5 jokes were returned (you'll see `"amount": 5` in the JSON).

At this stage, the response is stored, but it's still in raw format. To work with the data more easily, we need to load it into the Data Buffer.

## Loading the Response into the Data Buffer

The **Data Buffer** is a structured table where the API Engine parses and stores the response data in a way that's easy to work with. Each JSON object, array element, and key-value pair becomes a separate record in the data buffer.

### Step 1: Load the Response

1. On the API Message card, click the **Load Response** action (in the Manual Processing group).
2. The API Engine will parse the JSON response and create structured records in the Data Buffer table.
3. The **Message Status** will change to **DataBuffer Loaded**.

### Step 2: Open the Data Buffer

1. On the API Message card, navigate to **Related** > **Message Data Buffer** (or click the action from the navigation area).
2. The **API Data Buffers** page will open, showing all data buffer records for this message.

### Step 3: Understand the Data Buffer Structure

The Data Buffer page shows a hierarchical structure of the response data. Each row represents a piece of data from the JSON response.

You should see records similar to:

| Entry No. | Parent Entry No. | API Source Type | Name | Value | Data Type |
|-----------|------------------|-----------------|------|-------|-----------|
| 1 | 0 | Object | error | false | Boolean |
| 2 | 0 | Object | amount | 5 | Integer |
| 3 | 0 | Array | jokes | | Array |
| 4 | 3 | Object | (array element) | | Object |
| 5 | 4 | Object | category | Miscellaneous | Text |
| 6 | 4 | Object | type | single | Text |
| 7 | 4 | Object | joke | (joke text) | Text |
| ... | ... | ... | ... | ... | ... |

**Understanding the Structure:**

- **Entry No.:** Unique identifier for each data buffer record
- **Parent Entry No.:** Links child records to their parent (e.g., array elements to the array)
- **API Source Type:** Indicates if this is an Object, Array, or Value
- **Name:** The JSON key name (or index for array elements)
- **Value:** The actual data value (for simple values)
- **Data Type:** The data type (Text, Integer, Boolean, etc.)

### Step 4: Analyze the Jokes

1. Look for records where **Parent Entry No.** points to the `jokes` array (Entry No. 3 in the example above).
2. Each joke is a separate object within the array.
3. Expand (or filter to) one of the joke objects to see its properties:
   - `category`: Should show "Miscellaneous"
   - `type`: Single or twopart
   - `joke` or `setup`/`delivery`: The actual joke content
   - `id`: Unique joke ID
   - `safe`: Boolean indicating if it's safe
   - `lang`: Language code

### Step 5: Verify the Count

Count how many joke objects are in the data buffer. You should see exactly **5 jokes**, matching the value we specified in the `amount` parameter.

If you see 10 jokes instead of 5, the parameter wasn't correctly applied—double-check your function configuration and parameter key name.

## Understanding What We Achieved

Let's recap what we accomplished in this session:

### ✅ Multiple Runtime Parameters

We successfully used **two different types** of runtime parameters in a single API call:

1. **Parameter Record:** Retrieved the joke category from an Item Category table record
2. **Key-Value Parameter:** Specified the number of jokes using a simple key-value pair

### ✅ Complete URL Construction

Our final URL was dynamically constructed from multiple sources:

- **Credential:** Base URL (`https://v2.jokeapi.dev`)
- **Function:** Static path (`joke`)
- **Parameter Record:** Variable path (category name from Item Category)
- **Static Variable:** URL parameter tag (`safe-mode`)
- **Key-Value Parameter:** URL parameter (`amount=5`)

### ✅ Manual Message Lifecycle

We walked through the message lifecycle step-by-step:

1. **Initialized:** Created message and added parameters
2. **Request/Response:** Sent request and received response
3. **DataBuffer Loaded:** Parsed response into structured data

This manual process helps you understand what happens behind the scenes when you click **Execute**.

## Key Concepts Summary

### Parameter Types

| Type | Value Source | Best For | Example |
|------|-------------|----------|---------|
| **Static** | Hardcoded value | Values that never change | API version, fixed filters |
| **Parameter Record** | BC table record field | Existing BC data | Customer name, item category |
| **Parameter Value** | Key-value pair | User input, simple values | Quantity, date, search term |

### API Message Lifecycle Stages

| Stage | Status | Description | User Actions |
|-------|--------|-------------|--------------|
| 1 | Initialized | Message created | Add parameters, configure |
| 2 | Request | Request sent | Wait for response |
| 3 | Response | Response received | View response, load to buffer |
| 4 | DataBuffer Loaded | Data parsed | View buffer, process data |
| 5 | Complete | Data processed | Review results |

### Key Field: Static Name vs. Key

- **Static Name** (in API Variable): The parameter name expected by the external API (e.g., `amount`)
- **Key** (in API Parameter): The parameter name used to match the variable (must match Static Name)

Both must align for the parameter value to be correctly applied!

## Exercises

Apply what you've learned by completing the following exercises:

### Exercise 1: Change the Number of Jokes

1. Create a new API Message for the same function.
2. Add the required parameters, but this time set `amount` to `3`.
3. Send the message and verify that exactly 3 jokes are returned.

### Exercise 2: Try Different Categories

1. Create additional Item Category records for other Joke API categories:
   - Programming
   - Pun
   - Dark
   - Spooky
   - Christmas
2. Create new API Messages using different category parameters.
3. Compare the types of jokes returned for each category.

### Exercise 3: Test Without Amount Parameter

1. Create a new API Message.
2. Add only the category parameter (Table Record), but **do not** add an amount parameter.
3. Send the message and observe what happens.
4. Check the Data Buffer to see how many jokes were returned (the API's default).

### Exercise 4: Combine with Different Amounts

1. Create several messages using the same category but different amounts (1, 5, 10).
2. Compare the response times and data buffer sizes.
3. Document your findings.

### Exercise 5: Explore the Data Buffer

1. Pick one of your successfully executed messages.
2. Open the Data Buffer and examine the structure carefully.
3. Try to map out the parent-child relationships between records.
4. Identify which records represent the main response object vs. the jokes array vs. individual jokes.

## What's Next

In this workshop, we've successfully demonstrated how to use **Key-Value parameters** alongside **Parameter Record** variables, giving us complete flexibility in constructing dynamic API calls through the user interface.

However, we've only gone as far as viewing the data in the Data Buffer. We haven't actually **done anything** with the data yet—we haven't created any records in Business Central tables, we haven't displayed the jokes to users, and we haven't automated any processes.

In **Part 5**, we will finally start working with the response data by:

- **Understanding API Mappings**: Learn how to map response data to Business Central tables
- **Creating Target Tables**: Design tables to store jokes and related data
- **Configuring Mappings**: Set up field-level mappings from the Data Buffer to target tables
- **Processing Data**: Use the "Process Data Buffer" action to load data into BC tables
- **Validating Results**: Verify that data was correctly transferred and stored

Everything we've done so far has been preparation for this crucial step—taking external API data and integrating it into your Business Central environment.

Thank you for participating in Part 4 of the Joke API Workshop. Continue practicing with different parameter combinations, and we'll see you in Part 5!

## Additional Resources

- Joke API Documentation: [https://jokeapi.dev/](https://jokeapi.dev/)
- Joke API Video Part 4: [https://www.youtube.com/watch?v=WB6r4F1WlN0](https://www.youtube.com/watch?v=WB6r4F1WlN0)
- Joke API Video Playlist: [https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ)
- Suite Engine API Engine Documentation: [Contact Suite Engine for documentation]

## Quick Reference: Parameter Configuration

### To Create a Parameter Record Variable:
```
Value Processing Type: Parameter Record
Value Table No.: [Table Number]
Value Field No.: [Field Number]
```

### To Create a Parameter Value Variable:
```
Value Processing Type: Parameter Value
Variable Value Type: Key Value Text (or appropriate type)
Static Name: [parameter name for API]
```

### To Add a Table Record Parameter to a Message:
1. Click **Add New Parameter** > **Table Record**
2. Select table
3. Select record

### To Add a Key-Value Parameter to a Message:
1. Click **Add New Parameter** > **Key Value**
2. Set **Key** = parameter name
3. Set **Value** = desired value
