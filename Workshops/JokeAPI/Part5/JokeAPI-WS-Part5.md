# Joke API Workshop with API Engine - Part 5

## Introduction

Welcome to Part Five of our six-part JOKE API Workshop series. Up until now, we've focused on making API calls, constructing dynamic URLs, and retrieving data from external sources. We've learned how to view responses in their raw form and how the Data Buffer structures that data for processing. However, we haven't actually **done anything** with the data yet—it's been sitting in the Data Buffer, waiting.

In this crucial session, everything comes together. This is where the API Engine truly shines by bridging the gap between external APIs and your Business Central environment. We'll learn how to take the joke data we've been retrieving and automatically populate it into Business Central tables, making it available for use throughout the system.

By the end of this workshop, you'll understand how to use **API Mappings** to transform API responses into Business Central records, enabling true integration between external systems and your BC environment—all without writing a single line of code.

### Objectives

- Understand the purpose and structure of **API Mappings**.
- Learn how to use the **Update API Mapping** feature to auto-generate mapping skeletons from the Data Buffer.
- Configure different mapping **Source Types** (TableNode, RecordNode, Element).
- Map API response fields to Business Central table fields.
- Handle multiple data scenarios (single-part vs. two-part jokes).
- Configure the **Response Processing Method** to enable automatic data processing.
- Execute the complete API lifecycle from request to database update.
- Verify that data was correctly inserted or updated in BC tables.

## Requirements

Before participating in this workshop, you should have:

- Completed Parts One through Four of the series.
- Access to a SaaS Business Central environment with the Suite Engine API Engine Extension installed.
- Created and configured the JOKEAPI Set, Credentials, and Functions from previous workshops.
- Understanding of runtime parameters (both Parameter Record and Key-Value types).
- At least one successfully executed API Message with data loaded in the Data Buffer.
- Access to modify table data in Business Central (for testing purposes).

For additional reference, the detailed documentation for the Joke API endpoint is available at [https://jokeapi.dev/](https://jokeapi.dev/). You may also find our YouTube tutorial series helpful: [Joke API Video Tutorials](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ).

## Understanding API Mappings

### What Are API Mappings?

**API Mappings** are the configuration that tells the API Engine how to transfer data from the Data Buffer (where API responses are stored) into Business Central tables. They act as a bridge between external data structures and your BC database.

Think of mappings as a set of instructions:
- "Take the value from the `id` field in the API response and put it in the `Code` field of the Standard Text table."
- "For each joke in the `jokes` array, create a new record in the Standard Text table."
- "If a record with that Code already exists, update it; otherwise, insert a new one."

Without mappings, the Data Buffer is just a temporary storage area. With mappings, the API Engine knows exactly what to do with the data.

### Mapping Components

An API Mapping consists of several key elements:

| Component | Description | Purpose |
|-----------|-------------|---------|
| **Source ID** | The path/name of the data element in the API response | Identifies which data from the response to map |
| **Source Type** | The type of mapping node (Element, TableNode, RecordNode) | Defines how the engine should process this mapping |
| **Table No.** | The Business Central table number | Specifies where data should be written |
| **Field No.** | The Business Central field number | Specifies which field receives the data |
| **Parent Source ID** | The parent node in the hierarchy | Establishes nested relationships |
| **Sequence** | Processing order | Controls the sequence of operations |
| **IsKeyField** | Whether this field is a key | Determines if this is used for updates vs. inserts |

### Mapping Source Types

The API Engine supports several source types, each serving a different purpose:

#### 1. Element
- **Purpose:** Maps a single data value from the API response to a BC field
- **Example:** Map the `id` field from a joke object to the `Code` field in Standard Text
- **Use Case:** Individual fields like IDs, names, descriptions, dates, etc.

#### 2. TableNode
- **Purpose:** Represents the root object that corresponds to a BC table
- **Example:** The top-level response object that contains the overall structure
- **Use Case:** The outermost container for your data; usually marked as "Function Root Node"

#### 3. RecordNode
- **Purpose:** Represents a repeating structure (like an array) where each element becomes a BC record
- **Example:** The `jokes` array where each joke becomes one Standard Text record
- **Use Case:** Arrays or lists where each element should create/update a database record

#### 4. NormalizedNode
- **Purpose:** Represents a nested table relationship (parent-child tables)
- **Example:** If jokes had "tags" that went into a separate Joke Tags table
- **Use Case:** Complex structures with master-detail relationships

### The Mapping Hierarchy

Mappings follow a hierarchical structure that mirrors the API response:

```
TableNode (Function Root Node)
└── RecordNode (jokes array)
    ├── Element (id → Code field)
    ├── Element (joke → Description field)
    ├── Element (setup → Description field)
    └── Element (delivery → ignored/appended)
```

The **Parent Source ID** field links child mappings to their parent, creating this hierarchy.

## Reviewing the Data Buffer Structure

Before we create mappings, let's understand the structure of our API response by examining a previous message's Data Buffer.

### Step 1: Open a Previous API Message

1. Search for **API Messages** in Business Central.
2. Find a message where you successfully retrieved jokes (status should be **DataBuffer Loaded** or **Complete**).
3. Open the message card.

### Step 2: Navigate to the Data Buffer

1. From the API Message card, go to **Related** > **Message Data Buffer**.
2. The API Data Buffers page opens, showing the structured response data.

### Step 3: Analyze the Structure

Examine the data buffer records. You should see a structure similar to:

| Entry No. | Parent Entry | Name | Value | API Source Type | Path |
|-----------|-------------|------|-------|-----------------|------|
| 1 | 0 | error | false | Object | error |
| 2 | 0 | amount | 6 | Object | amount |
| 3 | 0 | jokes | | Array | jokes |
| 4 | 3 | | | Object | jokes[0] |
| 5 | 4 | category | Miscellaneous | Object | jokes[0].category |
| 6 | 4 | type | single | Object | jokes[0].type |
| 7 | 4 | joke | (joke text) | Object | jokes[0].joke |
| 8 | 4 | id | 94 | Object | jokes[0].id |
| ... | ... | ... | ... | ... | ... |
| 12 | 3 | | | Object | jokes[1] |
| ... | ... | ... | ... | ... | ... |

**Key Observations:**

1. **The jokes array** (Entry 3) is the parent of all individual joke objects
2. **Each joke** has a unique Entry No. and references its parent (Entry 3)
3. **Single-part jokes** have a `joke` field containing the full text
4. **Two-part jokes** have `setup` and `delivery` fields instead of `joke`
5. **The `id` field** is unique for each joke and makes a good key field
6. **The `type` field** tells us whether it's "single" or "twopart"

This structure will guide how we set up our mappings.

## Using the Update API Mapping Feature

The API Engine includes a powerful feature that can automatically generate a skeleton of mappings based on the data it sees in the Data Buffer. This saves significant time and reduces errors.

### Step 1: Open the Data Buffer

1. From your API Message card, navigate to **Related** > **Message Data Buffer**.
2. The API Data Buffers page opens with all data buffer records for this message.

### Step 2: Click Update API Mapping

1. On the API Data Buffers page, find the **Update API Mapping** action (it may be in the Actions ribbon or promoted actions).
2. Click **Update API Mapping**.
3. The API Engine analyzes the Data Buffer structure and generates mapping records.

**What Happens:**
- The engine examines the hierarchical structure in the Data Buffer
- It identifies objects, arrays, and individual values
- It creates a skeleton of mapping records with appropriate Source IDs and Parent Source IDs
- It sets preliminary Source Types based on the data structure

### Step 3: Confirmation

After the process completes, you'll see a confirmation message indicating how many mapping records were created or updated.

## Viewing and Understanding the Generated Mappings

Now let's examine the mappings that were created.

### Step 1: Navigate to API Mappings

1. Go back to the API Function card for your **GETJOKESBYCATEGORY** function.
2. Navigate to **Related** > **API Mappings** (or use the Actions ribbon).
3. The **API Mappings** page opens, showing all mapping records for this function.

### Step 2: Review the Mapping Structure

You should see records similar to:

| Sequence | Source ID | Parent Source ID | Source Type | Table No. | Field No. |
|----------|-----------|------------------|-------------|-----------|-----------|
| 10 | (root) | | TableNode | 0 | 0 |
| 20 | jokes | (root) | RecordNode | 0 | 0 |
| 30 | jokes[0].id | jokes | Element | 0 | 0 |
| 40 | jokes[0].category | jokes | Element | 0 | 0 |
| 50 | jokes[0].type | jokes | Element | 0 | 0 |
| 60 | jokes[0].joke | jokes | Element | 0 | 0 |
| 70 | jokes[0].setup | jokes | Element | 0 | 0 |
| 80 | jokes[0].delivery | jokes | Element | 0 | 0 |

**Note:** The exact Source IDs may vary based on your BC version and how the Data Buffer stores paths. The important thing is the hierarchical relationship and the Source Types.

**Understanding the Structure:**

1. **Root/TableNode** - The top-level response container
2. **jokes/RecordNode** - The array of jokes (each element becomes a record)
3. **jokes[0].xxx/Element** - Individual fields within each joke

The `[0]` indicates the first element in the array, but the mapping applies to all elements.

### Step 3: Identify What Needs Configuration

The generated skeleton has correct structure but needs:
1. ✅ **Source IDs** - Already set correctly
2. ✅ **Parent Source IDs** - Already set correctly
3. ✅ **Source Types** - Already set correctly
4. ❌ **Table Numbers** - Need to be specified (currently 0)
5. ❌ **Field Numbers** - Need to be specified (currently 0)
6. ❌ **Function Root Node** - Need to mark the root mapping

## Configuring the Target Table

For this workshop, we'll use the **Standard Text** table (Table 7) as our target. This table is simple and exists in all Business Central environments, making it perfect for demonstration.

**Important Note:** In a production scenario, you would create a custom "Joke" table with fields specifically designed for joke data (ID, Category, Type, Setup, Delivery, Joke, Language, Flags, etc.). For now, we'll work with what's available.

### Understanding the Standard Text Table

The Standard Text table has the following key fields:

| Field Name | Field No. | Type | Description |
|------------|-----------|------|-------------|
| Code | 1 | Code[20] | Primary key |
| Description | 2 | Text[100] | Text content |

We'll map:
- `jokes[0].id` → `Code` field (primary key)
- `jokes[0].joke` (or `setup`) → `Description` field

### Step 1: Verify the Table is Empty (Optional)

Before we start, let's check that the Standard Text table is empty so we can clearly see our test data.

1. Search for **Standard Texts** in Business Central.
2. The Standard Text page opens.
3. If there are existing records, note them so you can identify your test jokes later.
4. For this workshop, it's helpful if the table starts empty.

## Configuring the Mappings

Now we'll configure each mapping record to specify which BC table and fields should receive the data.

### Step 1: Configure the Root TableNode

1. On the API Mappings page, locate the root mapping (usually the first record, with no Parent Source ID).
2. Edit this record:
   - **Function Root Node:** Set to `True` (checkmark)
   - **Source Type:** Ensure it's set to `TableNode`
   - **Table No.:** Enter `7` (Standard Text table)
   - **Field No.:** Leave as `0` (TableNode doesn't map to a specific field)

**What This Does:**
- Marks this as the starting point for successful API call processing
- Indicates that the overall processing target is the Standard Text table

### Step 2: Configure the RecordNode (jokes array)

1. Locate the mapping for the `jokes` array (Source ID might be `jokes` or similar).
2. Edit this record:
   - **Source Type:** Ensure it's set to `RecordNode`
   - **Parent Source ID:** Should point to the root mapping's Source ID
   - **Table No.:** Enter `7` (Standard Text table)
   - **Field No.:** Leave as `0` (RecordNode doesn't map to a specific field)

**What This Does:**
- Tells the engine that for each element in the `jokes` array, create or update a record in table 7
- Establishes the iteration point—the engine will loop through all jokes

### Step 3: Configure the ID Element (Key Field)

1. Locate the mapping for the `id` field (Source ID might be `jokes[0].id` or `jokes.id`).
2. Edit this record:
   - **Source Type:** Ensure it's set to `Element`
   - **Parent Source ID:** Should point to the `jokes` RecordNode
   - **Table No.:** Enter `7` (Standard Text table)
   - **Field No.:** Enter `1` (Code field)

**What This Does:**
- Maps the joke ID from the API to the Code field in Standard Text
- The engine automatically detects that Code is a key field and marks **IsKeyField** as true
- This enables the engine to determine whether to insert (new ID) or update (existing ID)

### Step 4: Configure the Joke Element (Single-Part Jokes)

1. Locate the mapping for the `joke` field (Source ID might be `jokes[0].joke`).
2. Edit this record:
   - **Source Type:** Ensure it's set to `Element`
   - **Parent Source ID:** Should point to the `jokes` RecordNode
   - **Table No.:** Enter `7` (Standard Text table)
   - **Field No.:** Enter `2` (Description field)

**What This Does:**
- Maps the full joke text to the Description field
- This mapping applies to single-part jokes (where `type` = "single")

### Step 5: Configure the Setup Element (Two-Part Jokes)

1. Locate the mapping for the `setup` field (Source ID might be `jokes[0].setup`).
2. Edit this record:
   - **Source Type:** Ensure it's set to `Element`
   - **Parent Source ID:** Should point to the `jokes` RecordNode
   - **Table No.:** Enter `7` (Standard Text table)
   - **Field No.:** Enter `2` (Description field)

**What This Does:**
- Maps the setup part of two-part jokes to the Description field
- This mapping applies to two-part jokes (where `type` = "twopart")

**Important:** We're mapping both `joke` and `setup` to the same field (Description). This is intentional:
- Single-part jokes have only a `joke` field; the `setup` mapping will be ignored
- Two-part jokes have only `setup` and `delivery` fields; the `joke` mapping will be ignored
- The API Engine skips mappings where the source data doesn't exist

### Step 6: Handle Other Fields

For the remaining fields (`category`, `type`, `delivery`, etc.), you have options:

**Option A: Leave Unmapped**
- Simply don't configure Table No. and Field No. for these fields
- They will remain in the Data Buffer but won't be written to the table

**Option B: Map to Additional Fields**
- If you want to capture more data, you could use additional fields in Standard Text
- Or use the Extended Text Lines related table

For this workshop, we'll keep it simple and only map `id` to Code and `joke`/`setup` to Description.

## Configuring the Response Processing Method

Now that our mappings are configured, we need to tell the API Function to actually process the response using these mappings.

### Step 1: Open the API Function

1. Navigate back to the **API Functions** list.
2. Open your **GETJOKESBYCATEGORY** function (or the function you're working with).

### Step 2: Set the Response Processing Method

1. On the API Function card, locate the **Response** group (may be a FastTab or section).
2. Find the **Response Processing Method** field.
3. Change it from `None` (or blank) to **Multiple Records Update**.

**Response Processing Method Options:**

| Option | Description | Use Case |
|--------|-------------|----------|
| **None** | No automatic processing | When you only need to view responses, not store them |
| **Single Record Update** | Process response as a single record | APIs that return one object (e.g., get customer by ID) |
| **Multiple Records Update** | Process response as multiple records | APIs that return arrays/lists (e.g., get list of jokes) |

### Step 3: Save the Function

Close the API Function card to save your changes.

## Executing the Complete Lifecycle

Now we're ready to execute the full API call lifecycle, from creating the message to seeing the data in the Standard Text table.

### Step 1: Create a New API Message

1. From the API Functions page, select your **GETJOKESBYCATEGORY** function.
2. Navigate to **Related** > **API Messages**.
3. Click **+ New** to create a new message.

### Step 2: Add Runtime Parameters

Remember, our function requires two parameters:

**Parameter 1: Category (Table Record)**
1. In the Parameters section, click **Add New Parameter** > **Table Record**.
2. Select table `5722` (Item Category).
3. Choose a category like **Miscellaneous** or **Programming**.

**Parameter 2: Amount (Key-Value)**
1. Click **Add New Parameter** > **Key Value**.
2. Set:
   - **Key:** `amount`
   - **Value:** `6` (or any number you prefer)

### Step 3: Execute the Message

Instead of manually stepping through (Send → Load Response → Process Data Buffer), we can use the **Execute** button to run all steps automatically.

1. On the API Message card, click **Execute** (or **Actions** > **Execute**).
2. The API Engine will:
   - Build the request URL
   - Send the HTTP request
   - Receive the response
   - Load the response into the Data Buffer
   - Process the Data Buffer according to the mappings
   - Insert or update records in the Standard Text table

### Step 4: Review the Message Status

After execution completes:

1. Check the **Message Status**—it should show **Complete**.
2. Check the **HTTP Response Status Code**—should be **200**.
3. Review the **Query Request** to see the constructed URL.

If there are any errors, review the error messages and check:
- Are the mappings configured correctly?
- Is the Response Processing Method set?
- Did the API return data?

## Verifying the Results

Now let's verify that the jokes were actually written to the Standard Text table.

### Step 1: Open the Standard Text Page

1. Search for **Standard Texts** in Business Central.
2. The Standard Text page opens.

### Step 2: Find Your Jokes

You should see new records with:
- **Code:** Numeric values corresponding to joke IDs (e.g., 94, 105, 123)
- **Description:** Joke text (either the full joke or the setup line for two-part jokes)

For example:
- Code: `94`
- Description: `My mother said "You can be anything you want...`

### Step 3: Verify the Count

Count the records. The number of new jokes should match the `amount` parameter you specified (e.g., 6).

### Step 4: Review the Data Buffer

Go back to your API Message and view the Data Buffer to see the correlation between the buffer data and the Standard Text records.

1. From the API Message card, go to **Related** > **Message Data Buffer**.
2. Find a joke entry (e.g., Entry with `id` = 94).
3. Note the joke text in the Data Buffer.
4. Compare it to the Standard Text record with Code = `94`.
5. They should match (or the Standard Text may show the first 100 characters if truncated).

## Understanding Insert vs. Update Behavior

The **Multiple Records Update** processing method is smart about handling existing records.

### Test: Re-Execute the Same Message

1. Go back to your API Message (or create a new one with the same parameters).
2. Click **Execute** again.
3. Check the Standard Text table again.

**What You Should Observe:**
- No duplicate records are created
- If the API returned some of the same joke IDs, those records are **updated**, not inserted again
- Any new joke IDs create new records

### How It Works

1. The API Engine reads the joke `id` from the Data Buffer.
2. It checks if a Standard Text record with that `Code` already exists.
3. If yes → **Update** the existing record (Description may change if joke text changed).
4. If no → **Insert** a new record.

This is controlled by the **IsKeyField** flag on the mapping. When Field No. 1 (Code) was configured, the engine recognized it as a key field and enabled update/insert logic.

## Analyzing Two-Part Jokes

Let's examine how the system handles two-part jokes with `setup` and `delivery` fields.

### Step 1: Find a Two-Part Joke in the Data Buffer

1. Open the Data Buffer for your message.
2. Look for jokes where the `type` field = `twopart`.
3. Note the joke ID, setup, and delivery values.

### Step 2: Check the Standard Text Record

1. Open the Standard Text page.
2. Find the record with the Code matching the two-part joke ID.
3. Look at the Description field.

**What You Should See:**
- The Description contains the `setup` text only (because we mapped `setup` to Description).
- The `delivery` text is not included (because we didn't map it or mapped it to a field that doesn't exist).

**Why?**
- Two-part jokes don't have a `joke` field, so the `jokes[0].joke` → Description mapping is skipped.
- Two-part jokes have a `setup` field, so the `jokes[0].setup` → Description mapping is applied.
- The `delivery` text is ignored unless we configure a mapping for it.

### Improving the Mapping (Optional)

If you want to capture both setup and delivery, you could:

**Option 1: Concatenate**
- Use mapping transformation features (advanced) to combine setup + delivery.

**Option 2: Use Extended Text**
- Map delivery to a related Extended Text Lines table (requires additional configuration).

**Option 3: Create a Custom Table**
- In a real implementation, create a Joke table with separate Setup and Delivery fields (we'll discuss this in Part 6).

For now, we're keeping it simple with just the setup line.

## Key Concepts Summary

### API Mapping Source Types

| Source Type | Purpose | Example |
|-------------|---------|---------|
| **TableNode** | Root container linked to a BC table | Top-level response object |
| **RecordNode** | Array element that becomes a BC record | Each joke in jokes array |
| **Element** | Single data value mapped to a BC field | joke ID, joke text |
| **NormalizedNode** | Nested table relationship | Master-detail structures |

### Mapping Configuration Checklist

For each mapping record, ensure:
- [ ] **Source ID** is correctly identified from Data Buffer
- [ ] **Parent Source ID** links to correct parent (for hierarchy)
- [ ] **Source Type** matches the data structure
- [ ] **Table No.** specifies target BC table
- [ ] **Field No.** specifies target BC field (for Elements)
- [ ] **Function Root Node** is checked for the root TableNode
- [ ] **Sequence** provides logical processing order

### Response Processing Methods

| Method | When to Use | Result |
|--------|-------------|--------|
| **None** | View-only responses | Data stays in Data Buffer only |
| **Single Record Update** | One object returned | One BC record inserted/updated |
| **Multiple Records Update** | Array/list returned | Multiple BC records inserted/updated |

### The Complete Lifecycle

1. **Initialized** - Message created, parameters added
2. **Request** - URL built, request sent
3. **Response** - Response received and stored
4. **DataBuffer Loaded** - Response parsed into structured buffer
5. **Complete** - Mappings processed, data written to BC tables ✅

## Exercises

Apply what you've learned by completing the following exercises:

### Exercise 1: Retrieve Different Categories

1. Create new API Messages with different category parameters (Programming, Pun, Dark, etc.).
2. Execute each message.
3. Verify that jokes from different categories are added to the Standard Text table.
4. Confirm that duplicate joke IDs (same joke in different categories) update rather than create duplicates.

### Exercise 2: Retrieve Different Amounts

1. Create messages with different `amount` values (1, 3, 10).
2. Execute them and verify the correct number of records are added each time.
3. Check if the API has a maximum limit (try 100—it may return fewer).

### Exercise 3: Identify Two-Part Jokes

1. Review your Standard Text records.
2. Identify which ones are two-part jokes (they'll typically start with "Why" or a question).
3. Go back to the Data Buffer and find those joke records.
4. Examine the `delivery` field values that weren't mapped.
5. Consider how you might capture that data in a real implementation.

### Exercise 4: Test Update Behavior

1. Note a specific joke ID in the Standard Text table.
2. Manually edit the Description field for that record.
3. Execute the API function again (with a high amount to increase chances of getting the same joke).
4. Check if your manual changes are overwritten (they should be if the same joke is retrieved).

### Exercise 5: Create Mappings for Joke Categories

This is more advanced. The Joke API has an endpoint for retrieving available categories: `https://v2.jokeapi.dev/categories`

1. Create a new API Function called `GETJOKECATEGORIES`.
2. Set HTTP Method to `GET`.
3. Set API Path to `categories`.
4. Execute it and view the Data Buffer.
5. The response is a simple array of text values.
6. Create mappings to store categories in a BC table (perhaps Item Category or a custom table).
7. Configure Response Processing Method to Multiple Records Update.
8. Execute and verify categories are stored.

## Troubleshooting Common Issues

### Issue 1: Mappings Not Processing

**Symptoms:** Message completes but no records appear in Standard Text table.

**Possible Causes:**
- Response Processing Method not set (should be "Multiple Records Update")
- Function Root Node not checked on the root TableNode mapping
- Table No. or Field No. not configured on mappings

**Solution:** Review function configuration and mapping setup.

### Issue 2: Duplicate Records Created

**Symptoms:** Same joke appears multiple times with different Codes.

**Possible Causes:**
- IsKeyField not set correctly
- Mapping to wrong field as key
- Primary key field not mapped

**Solution:** Ensure the ID element is mapped to the Code field (Field No. 1) and verify IsKeyField is checked.

### Issue 3: Data Truncated

**Symptoms:** Joke text is cut off in the Description field.

**Possible Causes:**
- Description field (Text[100]) has a length limit
- Long jokes exceed the field capacity

**Solution:** This is expected for Text[100]. In a real implementation, use a larger field or Blob field.

### Issue 4: Two-Part Jokes Missing Delivery

**Symptoms:** Two-part jokes only show the setup, not the punchline.

**Possible Causes:**
- Delivery field not mapped
- Only setup is mapped to Description

**Solution:** This is by design in our simple example. To capture delivery, either concatenate in the mapping or use a custom table with separate fields.

## What's Next

Congratulations! You've successfully configured API Mappings and processed API response data into Business Central tables—all without writing a single line of code.

However, there's still room for improvement:

### Limitations of Our Current Setup

1. **Standard Text Table** - Not ideal for joke data; it's a generic table
2. **Single Field for Jokes** - We can't properly store two-part jokes
3. **Missing Data** - We're not capturing category, type, flags, language, etc.
4. **No User Interface** - Users can't easily browse or display jokes

### What's Coming in Part 6

In the final part of this series, we'll take everything to the next level by introducing **AL Code**:

- **Create a Custom Joke Table** - Design a proper table structure for joke data
- **Create Joke List and Card Pages** - Build user interfaces to view and manage jokes
- **Improve Mappings** - Map all joke fields to appropriate table fields
- **Add Business Logic** - Code to handle special cases, validations, and user actions
- **Build a Joke Display Page** - Create a page where users can view random jokes
- **Automate API Calls** - Use code to trigger API calls from user actions or schedules
- **Handle Errors Gracefully** - Implement error handling and logging

We'll transition from configuration-only to a production-ready solution that showcases the full power of the API Engine when combined with custom AL code.

Thank you for participating in Part 5 of the Joke API Workshop. Continue practicing with mappings and different API responses, and we'll see you in the final part!

## Additional Resources

- Joke API Documentation: [https://jokeapi.dev/](https://jokeapi.dev/)
- Joke API Video Part 5: [Part 5 Video Link]
- Joke API Video Playlist: [https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ)
- Suite Engine API Engine Documentation: [Contact Suite Engine for documentation]

## Quick Reference: Mapping Configuration

### Source Types
- **TableNode**: Root container → Set Function Root Node = True
- **RecordNode**: Array → Each element becomes a record
- **Element**: Single value → Maps to specific field

### Key Field Mapping
```
Source ID: jokes[0].id
Parent Source ID: jokes
Source Type: Element
Table No.: 7
Field No.: 1 (Code)
IsKeyField: True (automatic)
```

### Data Field Mapping
```
Source ID: jokes[0].joke
Parent Source ID: jokes
Source Type: Element
Table No.: 7
Field No.: 2 (Description)
IsKeyField: False
```

### Response Processing
- API Function → Response → Response Processing Method → Multiple Records Update
