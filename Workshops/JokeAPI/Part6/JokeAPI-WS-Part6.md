# Joke API Workshop with API Engine - Part 6

## Introduction

Welcome to Part Six of the JOKE API Workshop series. In Part Five, we explored API Mappings for handling multi-field record arrays, where each array element contains an object with multiple fields. We used the **Element** source type to map individual fields like `jokes[0].id` and `jokes[0].joke`.

Part Six introduces a different array pattern: **single-value arrays**. These arrays contain simple values (strings, numbers, dates) rather than objects with fields. The Joke API's categories endpoint demonstrates this pattern, returning a straightforward list: `["Programming", "Miscellaneous", "Dark", "Pun", "Spooky", "Christmas"]`.

For this pattern, we use the **Record Node Value** source type instead of Element. Understanding the distinction between these source types is essential for successful mapping configuration. This workshop builds on the concepts from Part 5, focusing on the key differences in data structure and mapping approach.

### Objectives

- Understand the difference between single-value arrays and multi-field record arrays
- Learn when to use Record Node Value vs. Element source types
- Configure mappings for simple array responses
- Map the same source value to multiple target fields for different purposes
- Apply insert/update logic using key fields

## Requirements

Before starting this workshop, you should have:

- Completed Parts 1-5 of the JOKE API Workshop series
- Access to a Business Central environment with the Suite Engine API Engine Extension installed
- A configured JOKEAPI set with credentials and functions from previous workshops
- Understanding of RecordNode and Element source types from Part 5

For additional reference, the Joke API documentation is available at [https://jokeapi.dev/](https://jokeapi.dev/), and our video tutorial series can be found at [Joke API Video Tutorials](https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ).

## Understanding Array Patterns

The API Engine must handle two distinct array structures:

### Pattern 1: Arrays of Objects (Part 5)

Multi-field record arrays contain objects with multiple properties:

```json
{
  "jokes": [
    {
      "id": 94,
      "category": "Miscellaneous",
      "joke": "My mother said...",
      "type": "single"
    },
    {
      "id": 231,
      "category": "Programming", 
      "setup": "Why do programmers prefer dark mode?",
      "delivery": "Because light attracts bugs.",
      "type": "twopart"
    }
  ]
}
```

**Key characteristics:** Each array element is an empty container with child fields beneath it. You map each field separately using **Element** source type:
- `jokes[0].id` → Target field
- `jokes[0].joke` → Target field
- Each child field requires its own mapping

### Pattern 2: Arrays of Values (Part 6)

Single-value arrays contain the data directly in each element:

```json
{
  "error": false,
  "categories": [
    "Programming",
    "Miscellaneous",
    "Dark",
    "Pun",
    "Spooky",
    "Christmas"
  ]
}
```

**Key characteristics:** Each array element IS the value itself. There are no child fields—the data exists at the element level. You map the element directly using **RecordNodeValue** source type:
- `categories[0]` → Code field
- `categories[0]` → Description field (same source, different target fields)

### Why Source Type Selection Matters

Selecting the incorrect source type will cause mapping failures:

| Scenario | Result |
|----------|--------|
| Use **Element** when you need **RecordNodeValue** | Engine searches for child fields that don't exist → No data mapped |
| Use **RecordNodeValue** when you need **Element** | Engine attempts to read value from an empty container → No data mapped |

**Decision Guide:**
- API returns `["value1", "value2"]` → Use **RecordNodeValue**
- API returns `[{"field": "value1"}, {"field": "value2"}]` → Use **Element**

## Reviewing the Item Category Table

For this workshop, we'll use the **Item Category** table (5722) as our target. While this is not ideal for production scenarios, it serves our educational purpose by providing the necessary fields without requiring custom table creation.

### Understanding the Target Fields

The Item Category table contains two key fields:

| Field Name | Field No. | Type | Behavior |
|------------|-----------|------|----------|
| Code | 1 | Code[20] | Primary key, automatically converts to UPPERCASE |
| Description | 2 | Text[100] | Preserves original casing |

**Important Note About Code Fields:**
Code-type fields in Business Central automatically convert all input to uppercase. This behavior is useful for standardization but can be problematic when working with case-sensitive APIs. The Joke API returns category names with specific casing (e.g., "Programming", "Miscellaneous"), and preserving this exact casing may be necessary for future API interactions.

**Solution Approach:**
We'll map the same source value to both fields:
- **Code field** → Used for insert/update logic (primary key)
- **Description field** → Preserves original casing for display and API compatibility

### Step 1: View Current Data

1. Search for **Item Categories** in Business Central
2. The Item Categories page displays existing demo data
3. Note the standard categories: CHAIR, DESK, MISC, TABLE
4. After completing this workshop, joke categories will be added to this table

## Creating the Get Joke Categories API Function

We'll now create an API function to retrieve the list of available joke categories from the endpoint.

### Step 1: Navigate to API Functions

1. Search for **API Functions** in Business Central
2. Locate your **JOKEAPI** set
3. Click **+ New** to create a new function

### Step 2: Configure the Function

Fill in the following details:

- **API Set Code:** JOKEAPI
- **Code:** GETJOKECATEGORIES
- **Description:** Get Joke Categories
- **HTTP Method:** GET
- **API Endpoint:** `categories`
- **Response Processing Method:** None (we will configure this after creating mappings)

**Note:** The complete URL will be `https://v2.jokeapi.dev/categories`. This endpoint requires no parameters and returns all available categories.

### Step 3: Save the Function

Close the API Function card to save your changes.

## Executing the API Call to Generate the Data Buffer

Before configuring mappings, we need to execute the API call at least once to populate the Data Buffer with the actual response structure. This Data Buffer will be used to automatically generate a mapping skeleton.

### Step 1: Create an API Message

1. From the API Functions page, select your **GETJOKECATEGORIES** function
2. Navigate to **Related** → **API Messages**
3. Click **+ New** to create a new message

### Step 2: Send the Request

Since we only need the Data Buffer for mapping configuration (and haven't set up processing yet), we'll execute the steps manually:

1. On the API Message card, click **Send Request** (not Execute)
2. The system sends the HTTP request and receives the response
3. Verify **Message Status** changes to **Response**
4. Verify **HTTP Response Status Code** shows **200** (success)

### Step 3: Load the Response

1. On the API Message card, click **Load Response**
2. The system parses the JSON and creates Data Buffer records
3. Verify **Message Status** changes to **DataBuffer Loaded**

**Do not click Process Data Buffer or Execute yet.** We have not configured mappings, and Response Processing Method is still set to None.

**Don't click Process Data Buffer or Execute yet.** No mappings configured, nothing will happen.

## Examine the Data Buffer Structure

This is the critical step: understanding what you're working with.

1. From the API Message → **Related** → **Message Data Buffer**
2. Look at the structure:

| Entry No. | Parent Entry | Name | Value | Path |
|-----------|-------------|------|-------|------|
| 1 | 0 | error | false | error |
| 2 | 0 | categories | | categories |
| 3 | 2 | | Programming | categories[0] |
| 4 | 2 | | Miscellaneous | categories[1] |
| 5 | 2 | | Dark | categories[2] |

**Notice:**
- Entry 2 (categories array) → No value, just container
- Entries 3-9 → **Name is empty**, **Value contains the category text**
- The value is AT the array element level, not in a child field

Compare to jokes (Part 5):
- Jokes: `jokes[0]` empty → `jokes[0].id` has value
- Categories: `categories[0]` HAS the value directly

That's why we need RecordNodeValue instead of Element.

## Generate Mapping Skeleton

Use the shortcut:

1. On Data Buffers page → **Update API Mapping**
2. API Engine analyzes structure and creates skeleton mappings
3. Go to Function → **Related** → **API Mappings**

You'll see generated records:

| Sequence | Source ID | Parent Source ID | Source Type | Table No. | Field No. |
|----------|-----------|------------------|-------------|-----------|-----------|
| 10 | (root) | | TableNode | 0 | 0 |
| 20 | categories | (root) | RecordNode | 0 | 0 |
| 30 | categories[0] | categories | Element | 0 | 0 |

The skeleton gives you structure but needs configuration:
- ❌ Table No. = 0 (not set)
- ❌ Field No. = 0 (not set)
- ❌ Source Type = Element (WRONG—needs RecordNodeValue)
- ❌ Function Root Node not checked

Let's fix it.

## Configure the Mappings

Now we fix the skeleton to actually work. We'll end up with three mappings:

### Mapping 1: Root Node (TableNode)

This tells the engine where to start processing:

| Field | Value | Why |
|-------|-------|-----|
| **Sequence** | 10 | Order matters—process root first |
| **Source ID** | `(root)` or blank | Top of JSON tree |
| **Parent Source ID** | blank | No parent—this IS the root |
| **Source Type** | `TableNode` | Entry point for processing |
| **Function Root Node** | ✅ Check this | Required for message processing |
| **Table No.** | 0 | Root doesn't target a table |
| **Field No.** | 0 | Root doesn't target a field |

**Function Root Node is critical.** Without it, the Process Data Buffer action won't know where to start.

### Mapping 2: Categories Array (RecordNode)

This represents the array container:

| Field | Value | Why |
|-------|-------|-----|
| **Sequence** | 20 | Process after root |
| **Source ID** | `categories` | Array name in JSON |
| **Parent Source ID** | `(root)` or blank | Child of root |
| **Source Type** | `RecordNode` | Array container |
| **Function Root Node** | blank | Only root gets this |
| **Table No.** | 5722 | Item Category table |
| **Field No.** | 0 | No specific field—this is the repeater |

**Why Table 5722 here?** This tells the engine "for each array element, insert/update a record in this table." The actual field mappings happen in the child mappings.

### Mapping 3a: Code Field (RecordNodeValue)

This maps the value to the Code field:

| Field | Value | Why |
|-------|-------|-----|
| **Sequence** | 30 | Process array elements |
| **Source ID** | `categories[0]` | Array element placeholder |
| **Parent Source ID** | `categories` | Child of array |
| **Source Type** | `RecordNodeValue` | **NOT Element**—value is AT the element |
| **Table No.** | 5722 | Same as parent |
| **Field No.** | 1 | Code field |
| **Is Key Field** | ✅ Check this | Used for insert/update logic |

**The [0] is a placeholder.** Engine processes all elements, not just the first one.

### Mapping 3b: Description Field (RecordNodeValue)

Duplicate the Code mapping, change the field:

1. On API Mappings page → Select the Code field mapping (Sequence 30)
2. **Actions** → **Duplicate** (or copy/paste if available)
3. Edit the new record:
   - **Sequence:** 40 (must be unique)
   - **Field No.:** 2 (Description field)
   - **Is Key Field:** ❌ Uncheck

Now you have both fields mapped to the same source:
- `categories[0]` → Field 1 (Code) - Key field, uppercases
- `categories[0]` → Field 2 (Description) - Preserves casing

**Why map same value twice?** Code field drives insert/update logic. Description field preserves exact casing for future use.

## Update Response Processing Method

Now that mappings exist, tell the function to use them:

1. Go back to API Function **GETJOKECATEGORIES**
2. **Response Processing Method:** Change from `None` to `DataBuffer Insert-Update`
3. Close to save

**What this does:** When you execute the function, it will:
1. Send request
2. Load response into Data Buffer
3. Process Data Buffer using your mappings
4. Insert/update Item Category records

## Test the Complete Flow

Time to see it work:

1. **API Functions** → Select GETJOKECATEGORIES
2. **Related** → **API Messages** → Create new message (or reuse existing)
3. **Execute** (not just Send Request this time)
4. Check **Message Status** → Should show `Processed`
5. Check **HTTP Response Status Code** → Should show `200`

If something fails, check **Message Errors** or **Related** → **Message Logs**.

## Verify the Results

Check if categories were actually created:

1. Search **Item Categories** in BC
2. You should now see:
   - CHAIR (existing demo data)
   - CHRISTMAS (new from API)
   - DARK (new from API)
   - DESK (existing demo data)
   - MISCELLANEOUS (new from API)
   - PROGRAMMING (new from API)
   - PUN (new from API)
   - SPOOKY (new from API)
   - TABLE (existing demo data)

**Notice:** Code field shows uppercase. Open a record and check Description—should preserve original casing like "Programming".

**If you run it again:** No duplicates will be created. The Is Key Field setting means the engine does upsert logic:
- Record exists with matching Code → Update Description
- Record doesn't exist → Insert new

## What Just Happened (Data Buffer Flow)

Let's trace one category through the whole pipeline:

**1. API Response:**
```json
{
  "error": false,
  "categories": ["Programming", "Miscellaneous", "Dark"]
}
```

**2. Data Buffer Structure:**
- Entry 1: error = false (ignored, no mapping)
- Entry 2: categories array (RecordNode, Table 5722)
- Entry 3: categories[0] = "Programming" (RecordNodeValue)
- Entry 4: categories[1] = "Miscellaneous" (RecordNodeValue)
- Entry 5: categories[2] = "Dark" (RecordNodeValue)

**3. Processing (Mapping Execution):**
- Root mapping: Start processing
- categories RecordNode: Target Table 5722, begin iteration
- For Entry 3 (Programming):
  - Mapping 3a: categories[0] → Field 1 (Code) → "PROGRAMMING" (uppercased)
  - Mapping 3b: categories[0] → Field 2 (Description) → "Programming" (preserved)
  - Check: Does record with Code="PROGRAMMING" exist?
    - No → Insert new record
    - Yes → Update Description field
- Repeat for entries 4, 5...

**4. Result:**
- New Item Category records created/updated
- Code field: PROGRAMMING (uppercase)
- Description field: Programming (original case)

This is why RecordNodeValue matters—we're reading the value directly from the array element, not from a child field.

## Common Issues and Fixes

### Issue: No Records Created

**Symptoms:**
- Message Status: Processed
- HTTP 200
- But Item Categories unchanged

**Check:**
1. **Response Processing Method** set to `DataBuffer Insert-Update`? (Most common mistake)
2. Root mapping has **Function Root Node** checked?
3. RecordNode mapping has correct **Table No. 5722**?
4. At least one child mapping exists with actual field numbers?

### Issue: Mappings Generated as Element, Not RecordNodeValue

**Symptoms:**
- Update API Mapping created Element type for categories[0]
- No data inserted into fields

**Fix:**
1. Manually change Source Type from `Element` to `RecordNodeValue`
2. This happens because Data Buffer shows API Source Type as "Object" even for value-bearing array elements

### Issue: Description Field Uppercased

**Symptoms:**
- Both Code and Description show "PROGRAMMING"
- Original casing lost

**Cause:**
- You mapped to Code field twice, or
- Description field somehow inheriting Code field behavior (rare)

**Fix:**
- Check Mapping 3b: Field No. should be 2 (Description), not 1 (Code)
- Field 2 is Text[100], doesn't uppercase

### Issue: Records Update Instead of Insert Every Time

**Symptoms:**
- First run works, subsequent runs show "Processed" but no new records
- Expected 7 categories, still only seeing 4

**Actually:** This is correct behavior if Is Key Field is set. Engine does upsert:
- Existing records updated (Description field refreshed with latest API values)
- Missing records inserted

**To verify:**
1. Manually delete one category (e.g., PROGRAMMING)
2. Re-run Execute
3. PROGRAMMING should be recreated

### Issue: Duplicate Records Created

**Symptoms:**
- Multiple records with same Code value
- PROGRAMMING appears 3 times

**Cause:**
- Is Key Field not checked on Code field mapping
- Without key field, engine always inserts, never updates

**Fix:**
1. Edit Mapping 3a (Code field)
2. Check **Is Key Field**
3. Manually delete duplicates from Item Categories
4. Re-run Execute

## Understanding the Complete Mapping Configuration

Let's review the complete mapping structure and processing flow:

### Mapping Hierarchy

```
TableNode (root) - Function Root Node ✓
└── RecordNode (categories) - Table 5722
    ├── RecordNodeValue (categories[0]) → Field 1 (Code) - IsKeyField ✓
    └── RecordNodeValue (categories[0]) → Field 2 (Description)
```

### Processing Flow

When the API Engine processes this configuration:

1. Engine starts at root mapping (Function Root Node = True indicates the starting point)
2. Identifies the categories array (RecordNode, targets Table 5722)
3. Iterates through each array element
4. For each category value (e.g., "Programming"):
   - Maps to Code field → "PROGRAMMING" (automatically uppercased by field type)
   - Maps to Description field → "Programming" (casing preserved)
   - Checks if a record with Code="PROGRAMMING" exists
   - Inserts new record if not found, updates existing record if found

### Comparison: Part 5 vs Part 6

**Part 5 (Multi-Field Records):**
```
jokes[0]             (empty container)
  ├── jokes[0].id    (contains value)
  └── jokes[0].joke  (contains value)
```
Use **Element** source type for child fields.

**Part 6 (Single-Value Arrays):**
```
categories[0] = "Programming" (value exists at this level)
```
Use **RecordNodeValue** source type for the element itself.

This structural difference determines the correct source type selection.

## What's Next

Congratulations on completing Part Six of the JOKE API Workshop series. You have now learned two essential mapping patterns:
- **Multi-field record arrays** (Part 5) using Element source type
- **Single-value arrays** (Part 6) using Record Node Value source type

These two patterns address the majority of API response structures encountered in real-world scenarios.

### Current Limitations

While we have accomplished significant functionality, the current implementation has limitations due to our use of standard Business Central tables:

1. **Field Constraints** - Standard Text and Item Category tables lack the fields needed to capture all joke data properties
2. **Data Structure** - These tables are not designed for joke or category data management
3. **Relationships** - We cannot properly establish relationships between jokes and their categories
4. **Business Logic** - No ability to add validation, calculations, or custom behavior
5. **User Experience** - No dedicated interface for viewing or managing joke data

### Future Workshop Topics

In upcoming workshops, we will move beyond configuration-only solutions and introduce **AL Code development**:

#### Custom Table Design
- Create dedicated **Joke** and **Joke Category** tables with appropriate fields
- Implement proper primary keys and relationships
- Add fields for all joke properties (ID, Category, Type, Flags, etc.)

#### Custom Pages
- **Joke List Page** - Browse and manage jokes
- **Joke Card Page** - View and edit individual jokes
- **Joke Category List Page** - Manage category data
- **Random Joke Display** - Interactive joke viewing with actions

#### Enhanced Business Logic
- Trigger API calls from user actions
- Schedule automated data refreshes
- Implement error handling and validation
- Add notifications and user feedback

### Continue Practicing

Before advancing to code-based workshops:
1. Experiment with additional API endpoints at [https://jokeapi.dev/](https://jokeapi.dev/)
2. Practice identifying Element vs. Record Node Value patterns in different response structures
3. Test edge cases and error scenarios
4. Review Parts 1-6 to reinforce your understanding

Thank you for participating in Part 6 of the Joke API Workshop. You have completed the foundational configuration-based training and are prepared for the next phase of custom AL development.

Business Logic:
- Trigger API calls from user actions
- Schedule daily refreshes
- Handle errors gracefully
- Data validation

The API Engine shines when you combine configuration (what we've done) with AL code (what's next). Configuration gives you speed. Code gives you power.

**Before Moving On:**
1. Try different JokeAPI endpoints
2. Practice recognizing Element vs RecordNodeValue patterns
3. Test edge cases (empty responses, errors, etc.)

That's Part 6. You're ready for custom development now.

## Quick Reference

**RecordNodeValue Decision:**
- Array contains `["value1", "value2"]` → RecordNodeValue
- Array contains `[{"field": "value1"}, {"field": "value2"}]` → Element

**Mapping Pattern:**
```
Root: Function Root Node ✓, TableNode, Table 5722
Array: RecordNode, Table 5722
Values: RecordNodeValue, same Source ID, different Field No.
```

**Response Processing Methods:**
- None → Data Buffer only, no table updates
- Single Record Update → One object from API
- Multiple Records Update → Array from API

**Common Issues:**
1. No records created → Check Response Processing Method
2. Empty Description → Missing second mapping
3. Source not found → Wrong source type (Element vs RecordNodeValue)
4. Duplicates → IsKeyField not checked
5. All uppercase → Both mappings target Code field instead of Code + Description

**Resources:**
- JokeAPI: https://jokeapi.dev/
- Video Playlist: https://www.youtube.com/playlist?list=PLKxx1u9Yh-cQXcJURIWiAQAwePW70o3VQ
