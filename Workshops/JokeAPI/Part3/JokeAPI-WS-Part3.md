# Joke API Workshop with API Engine - Part 3

## Introduction

Welcome to Part Three of our six-part JOKE API Workshop series. In the previous sessions, we learned how to create API sets, credentials, and functions, and we explored how to construct URLs using static values for paths and parameters. While this approach works well for fixed scenarios, real-world applications often require dynamic behavior based on runtime conditions.

This session marks an important transition in our workshop series. We will move from static URL construction to dynamic URL construction using variables and parameters. You will learn how to make API calls where the values are determined at runtime rather than being hardcoded. This capability is essential for building flexible, production-ready API integrations.

### Objectives

- Create API variables that allow URLs to be constructed with values provided at runtime.
- Understand the concept of API parameters and how they provide variable values during API execution.
- Learn to provide API parameters through the user interface (without writing code).
- Explore the lifecycle of an API message and understand each stage of the request-response process.
- Execute API functions step-by-step to gain deeper insight into how the API Engine processes messages.

## Requirements

Before participating in this workshop, you should have:

- Completed Part One and Part Two of the series.
- Access to a SaaS Business Central environment with the Suite Engine API Engine Extension installed.
- Created the API Set, Credentials, and Functions from the previous workshops.
- Basic understanding of URL construction, including Base URL, Path, and Query Parameters.

For additional reference, the detailed documentation for the Joke API endpoint is available at [https://jokeapi.dev/](https://jokeapi.dev/). You may also find our YouTube tutorial series helpful: [Joke API Video Tutorials](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ).

## Review of Existing API Functions

Before we begin creating dynamic functions, let's review what we've built so far in the previous workshops. We have created several API functions, each serving a specific purpose:

- **GETTENSAFEJOKE** - Retrieves 10 safe jokes from a random category (using "Any").
- **GETTENPROGJOKE** - Retrieves 10 jokes from the Programming category.
- **GETONESAFEJOKE** - Retrieves a single safe joke from a random category.
- **GETJOKECATEGORIES** - Retrieves all available joke categories from the API.

In each of these functions, we used **static values** for the URL path and parameters. For example:

- The URL Path used a static value of `Any` for the category.
- The URL Parameters used static values like `safe-mode` and `amount=10`.

While this approach is straightforward and easy to understand, it has a significant limitation: you cannot change these values without modifying the function configuration. In this workshop, we will overcome this limitation by using **runtime variables**.

## Understanding API Variables vs. Static Values

When constructing URLs in the API Engine, there are two approaches:

1. **Static Values** - Values that are hardcoded into the function and never change. These are what we've used in Parts 1 and 2.
2. **Variable Values** - Values that are determined at runtime, allowing for dynamic behavior based on parameters provided during execution.

The URL construction process in the API Engine follows this pattern:

`{Base URL} + {Static Path} + {Variable Path} + {Query Parameters}`

For example, in our previous workshops:

- **Base URL**: `https://v2.jokeapi.dev` (defined in the API Credential)
- **Static Path**: `joke` (defined in the function)
- **Variable Path**: `Any` (previously static, will now be dynamic)
- **Query Parameters**: `safe-mode&amount=10` (previously static)

By using variables, we can replace the static "Any" category with a category chosen at runtime, making our API function much more flexible.

## Create a New Function with Runtime Variables

Now we'll create a new API function that uses a runtime variable for the joke category. Instead of always requesting jokes from "Any" category, we'll allow the category to be specified when the function is executed.

### Step 1: Copy an Existing Function

1. Navigate to the **API Functions** list for the JOKEAPI set.
2. Select the **GETTENSAFEJOKE** function.
3. Go to **Actions** > **Copy API Function**.
4. Enter the following details for the copy operation:
   - Target API Set Code: `JOKEAPI`
   - Target Function Code: `GETJOKESBYCATEGORY`
   - Data To Copy:
     - General Function Information: `True`
     - API Variable: `True`
     - API Mapping: `True`
     - Credential: `True`

### Step 2: Update Function Description

1. Open the newly created **GETJOKESBYCATEGORY** function.
2. Update the description to: `Retrieves 10 safe jokes from a category provided at runtime.`

### Step 3: Configure the Variable Path for Runtime

Now we need to modify the URL Path variable to accept a value from a parameter instead of using a static value.

1. In the **GETJOKESBYCATEGORY** function, navigate to **API Variables** > **URL Path**.
2. Locate the **Category** variable that was copied from the original function.
3. Edit the **Category** variable with the following changes:
   - Name: `Category`
   - Value Processing Type: `Parameter Record`
   - Parameter Table No.: `5722` (Item Category table)
   - Parameter Field No.: `3` (Description field)
   - Undefined Variable Value: `undefined`

**Why are we using the Item Category table?**

Since we haven't yet created custom tables for joke categories (this will come in later parts of the series), we need to use an existing Business Central table. The Item Category table is a good choice because:

- It already exists in all Business Central environments.
- It has a Description field that allows lowercase text.
- We can easily add custom categories for testing.

**Why use the Description field instead of Code?**

The Code field in Business Central is typically stored in uppercase. However, many APIs, including the Joke API, are **case-sensitive**. The Joke API expects category names like "Programming" or "Miscellaneous" (with proper casing), not "PROGRAMMING" or "MISCELLANEOUS". By using the Description field, we can maintain the exact casing required by the API.

**What is the Undefined Variable Value?**

This setting determines what value should be used if the variable cannot be resolved at runtime (e.g., if no parameter is provided). Setting it to `undefined` will help us see clearly when a parameter is missing.

### Step 4: Prepare Test Data in Item Categories

Before we can test our function, we need to add some joke categories to the Item Category table.

1. In Business Central, search for **Item Categories**.
2. Note that the standard Cronus database already includes a category with Description **Miscellaneous**, which matches one of the Joke API categories.
3. Create a new Item Category record:
   - Code: `PROG`
   - Description: `Programming`
4. Create another Item Category record:
   - Code: `MISC`
   - Description: `Misc`
5. You can add other categories that match the Joke API categories (Any, Programming, Miscellaneous, Dark, Pun, Spooky, Christmas).

## Executing Without Parameters (Understanding the Error)

Now let's see what happens when we execute this function without providing a parameter value.

1. From the **API Functions** list, select the **GETJOKESBYCATEGORY** function.
2. Click **Execute** from the Home ribbon.
3. When prompted, click **Yes** to open the API Message.

### Analyzing the Failed Request

When you review the API Message, you'll notice:

- The HTTP Response Code indicates an error (likely 400 - Bad Request).
- In the **Query Request** section, you'll see the URL that was actually sent to the Joke API.
- The category portion of the URL will show `undefined` or be missing entirely.
- The **Response** section will contain an error message from the Joke API explaining that the required category parameter was not provided correctly.

**Why did this happen?**

We configured the function to use a variable from a parameter record, but we never actually provided that parameter during execution. When we clicked **Execute**, the API Engine immediately tried to send the request without waiting for parameter input. Since no parameter was available, the variable couldn't be resolved, and the API call failed.

This demonstrates an important concept: when using runtime variables, you must provide the corresponding parameters before the API call is made.

## Understanding the API Message Lifecycle

Before we learn how to provide parameters properly, it's important to understand the lifecycle of an API message in the SuiteEngine API Engine. Every API call goes through the following stages:

### Stage 1: Initialization

- An API Message record is created.
- The API Function to execute is specified.
- Any required API Parameters are attached to the message.

### Stage 2: Request Building

- The API Engine constructs the complete URL using the base URL, static paths, variable paths, and parameters.
- Any request payload (for POST/PUT/PATCH operations) is prepared.
- Headers and authentication are configured.

### Stage 3: Request Sending

- The HTTP request is sent to the external API endpoint.
- The API Engine waits for the response.

### Stage 4: Response Retrieval

- The HTTP response is received from the external API.
- The response status code, headers, and body are captured.

### Stage 5: Response Processing

- The response data is parsed (JSON, XML, etc.).
- Any configured mappings are applied (we'll cover this in later parts).
- The message status is updated to reflect the outcome.

**Two Ways to Execute**

Up until now, we've been using the **Execute** button, which runs through all five stages automatically without pausing. This is convenient for simple scenarios, but it doesn't give us the opportunity to provide parameters.

For scenarios requiring runtime parameters, we need to use the **API Messages** page, which allows us to:

1. Create a new message (Initialization).
2. Attach parameters to the message.
3. Manually send the message (Request Building, Sending, Retrieval, and Processing).

This step-by-step approach gives us full control over the message lifecycle.

## Providing Parameters via the User Interface

Now let's execute our function the correct way—by creating an API Message and providing the parameter before sending.

### Step 1: Navigate to API Messages

1. From the **API Functions** list, select the **GETJOKESBYCATEGORY** function.
2. Navigate to **Related** > **API Messages**.
   
Alternatively, you can search for **API Messages** directly in Business Central and filter to your function.

### Step 2: Create a New API Message

1. On the **API Messages** page, click **+ New** to create a new message.
2. The **API Function Code** should automatically populate with `GETJOKESBYCATEGORY` since you navigated from that function. If not, manually select it.
3. Do not click **Send** yet—we need to add parameters first.

### Step 3: Add API Parameters

At this stage, the API Message has been initialized, but it hasn't been sent yet. This is the perfect time to attach our parameter.

1. On the API Message card, scroll down to the **Parameters** section (you may need to expand the FactBox or navigate to a related page depending on your BC version).
2. Click **Add New Parameter** or navigate to **Related** > **API Parameters**.
3. On the API Parameters page, create a new record with the following details:
   - Parameter Type: `Table Record`
   - Table No.: `5722` (Item Category)
   - Look up and select a record:
     - Choose the Item Category with Description **Miscellaneous** (or **Programming** if you prefer).

**Important Note:**

Business Central stores this parameter as a record reference, linking to the specific Item Category record you selected. When the API call is made, the API Engine will retrieve the value from the Description field of this record and use it in the URL construction.

### Step 4: Verify the Parameter

Before sending, verify that your parameter is correctly attached:

1. Return to the API Message card.
2. In the Parameters section, you should see one parameter listed.
3. Confirm it references the correct Item Category record.

## Executing the API Message Step-by-Step

Now that we have our API Message with the parameter attached, let's walk through the message lifecycle manually. This will help reinforce your understanding of each stage.

### Step 1: Send the Request

1. From the API Message card, click **Send** (or **Process** depending on your version).
2. The API Engine will now:
   - Build the request URL using the parameter value.
   - Send the HTTP request to the Joke API.
   - Receive the response.
   - Update the message status.

### Step 2: Review the Message Status

After the send operation completes:

1. Check the **Status** field on the API Message card. It should now show **Response** (indicating the request was sent and a response was received).
2. Check the **HTTP Response Code** field. It should show **200**, indicating success.
3. Check the **Duration** field to see how long the API call took.

### Step 3: Analyze the Request

Let's verify that our parameter was correctly used in the URL construction.

1. On the API Message card, locate the **Query Request** section (or click **Related** > **Request** depending on your version).
2. Examine the full URL that was sent to the Joke API.
3. You should see something like:
   ```
   https://v2.jokeapi.dev/joke/Miscellaneous?safe-mode&amount=10
   ```
4. Notice that:
   - The category portion shows `Miscellaneous` (the value from the Item Category Description field).
   - The query parameters include `safe-mode` and `amount=10`.
   - The URL is properly formatted and complete.

### Step 4: Review the Response

Now let's look at the jokes we received.

1. On the API Message card, navigate to the **Response** section or click **Related** > **Message Data Buffer**.
2. You should see 10 joke records in the data buffer.
3. Examine a few of the jokes and verify:
   - All jokes have `"category": "Miscellaneous"` in their JSON structure.
   - All jokes are marked as safe (no flags field or flags set to safe).
   - There are exactly 10 jokes (matching our amount parameter).

**Success!**

You've successfully executed an API call using a runtime variable. Instead of hardcoding the category, we provided it dynamically through a parameter, and the API Engine correctly constructed the URL and retrieved the appropriate jokes.

## Testing with Different Categories

To further reinforce the concept, let's execute the function again with a different category.

### Option 1: Create a New Message with a Different Parameter

1. Navigate back to the **API Messages** page.
2. Create another new message for the **GETJOKESBYCATEGORY** function.
3. Add a parameter, but this time select the **Programming** Item Category.
4. Click **Send**.
5. Review the response and verify you received 10 programming jokes.

### Option 2: Re-execute Using Quick Execute

If you've added multiple Item Categories, you can also:

1. Go to the **API Functions** list.
2. Select **GETJOKESBYCATEGORY**.
3. Click **Execute** (this will fail, but shows you the difference).
4. Compare this to the manual process with parameters.

## Key Concepts Summary

Before moving on, let's summarize the key concepts covered in this workshop:

### Static vs. Variable Values

- **Static Values**: Hardcoded in the function configuration, cannot change at runtime.
- **Variable Values**: Determined at runtime based on parameters, allowing dynamic behavior.

### Value Processing Types

When configuring an API variable, you can choose different value processing types:

- **Static**: A fixed value that never changes.
- **Parameter Record**: A value retrieved from a specific field in a table record provided as a parameter.
- **Parameter Value**: A simple value provided directly as a parameter (we'll explore this in the next part).

### API Message Lifecycle

1. **Initialization**: Message created, function specified, parameters attached.
2. **Request Building**: URL and payload constructed using variables and parameters.
3. **Request Sending**: HTTP request sent to the API.
4. **Response Retrieval**: HTTP response received.
5. **Response Processing**: Response parsed and processed.

### Providing Parameters

- **Execute Button**: Runs the entire lifecycle immediately without pausing for parameter input.
- **Manual Message Creation**: Allows you to create a message, attach parameters, then send—giving full control over the process.

### Case Sensitivity

Many APIs are case-sensitive. When selecting fields to use for parameter values, ensure the field maintains the correct casing required by the API. In our example, we used the Description field instead of Code because Business Central's Code fields are typically uppercase.

## Exercise

Apply what you've learned in this session by completing the following exercise:

### Exercise 1: Create a Function for a Specific Number of Jokes

Currently, our **GETJOKESBYCATEGORY** function always retrieves 10 jokes. Your task is to create a modified version that retrieves a variable number of jokes based on a runtime parameter.

**Steps:**

1. Copy the **GETJOKESBYCATEGORY** function to a new function called **GETJOKESBYCAT_VAR**.
2. Keep the Category parameter as a Parameter Record (Item Category).
3. Modify the **amount** URL parameter to use a **Parameter Value** instead of a static value.
4. Test the function by creating an API Message and providing both:
   - A Category parameter (Item Category record)
   - An Amount parameter (a numeric value)

**Hint**: You'll need to research how to add a Parameter Value in addition to the Parameter Record. This will prepare you for the next workshop where we'll cover this in detail.

### Exercise 2: Explore Different Categories

1. Add more Item Categories to match other Joke API categories:
   - Dark
   - Pun
   - Spooky
   - Christmas
2. Execute the **GETJOKESBYCATEGORY** function with each category.
3. Compare the types of jokes returned by each category.

### Exercise 3: Analyze Failed Requests

1. Create an API Message for **GETJOKESBYCATEGORY**.
2. Add a parameter with an Item Category that has an invalid description (e.g., "InvalidCategory").
3. Send the message and analyze the error response from the Joke API.
4. Document what error message is returned and why.

## What's Next

In this workshop, you've learned how to use **Parameter Record** variables to make your API calls dynamic. However, we still have some limitations:

- The amount of jokes is still hardcoded at 10.
- We can only use values from existing Business Central tables.
- We need to create Item Categories just to store joke category names (which isn't ideal).

In **Part 4**, we will explore:

- **Key-Value Parameters**: Learn how to provide simple values (like the number of jokes) directly without needing a table record.
- **Multiple Parameters**: Combine different types of parameters in a single API call.
- **User Input**: Explore ways to prompt users for parameter values.
- **Parameter Validation**: Ensure parameters meet API requirements before sending requests.

Everything we've learned so far can be done without writing any code. However, in later parts of the series, we'll introduce AL code to create a polished, production-ready solution for managing jokes in Business Central.

Thank you for participating in Part 3 of the Joke API Workshop. Continue practicing with runtime variables and parameters, and we'll see you in Part 4!

## Additional Resources

- Joke API Documentation: [https://jokeapi.dev/](https://jokeapi.dev/)
- Joke API Video Part 3: [https://www.youtube.com/watch?v=65tAGBN-IrY](https://www.youtube.com/watch?v=65tAGBN-IrY)
- Joke API Video Playlist: [https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ)
- Suite Engine API Engine Documentation: [Contact Suite Engine for documentation]
