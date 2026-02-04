Generate a UiPath workflow file (.xaml) with the following specifications.

**Project and File Name:**
- Project Name: UiPathDemo_GITIntegration
- File Name: Process.xaml

**Packages & Dependencies:**
Ensure the workflow references these packages:
- `UiPath.System.Activities` (version 25.12.2 or higher)
- `UiPath.Excel.Activities` (version 2.11.4 or higher)
- `UiPath.UIAutomation.Activities` (version 21.10.10 or higher)

**Workflow Arguments:**
Define the following arguments for the workflow:
1.  `in_Config`:
    -   Direction: In
    -   Type: `System.Collections.Generic.Dictionary<System.String, System.Object>`
    -   Purpose: Holds configuration data, including the Orchestrator queue name.
2.  `InputDT`:
    -   Direction: In
    -   Type: `System.Data.DataTable`
    -   Purpose: The input table containing records to be added to the queue.
3.  `UploadComplete`:
    -   Direction: InOut
    -   Type: `System.Boolean`
    -   Purpose: A flag that will be set to `True` upon successful completion.

**Workflow Variables:**
Define the following variables scoped to the main sequence:
1.  `OnUsCheck`: Type `System.String`
2.  `CashIn`: Type `System.String`
3.  `NotOnUsCheck`: Type `System.String`

**Workflow Activity Structure:**

1.  **Create a root `Sequence` activity.**
    -   Set its `DisplayName` to "Data Dispatcher Process".

2.  **Inside the root Sequence, add a `For Each Row in Data Table` activity.**
    -   `DisplayName`: "Iterate Through Input Data"
    -   `DataTable` property: Set to the argument `InputDT`.
    -   The loop variable for the current row should be named `row`.

3.  **Inside the `Body` of the `For Each Row` loop, add the following activities in order:**

    a. **Add a `Try Catch` activity** to handle potential errors for each row.

    b. **Inside the `Try` block of the `Try Catch` activity:**
        i. **Add an `Assign` activity.**
            -   `DisplayName`: "Extract CashIn"
            -   `To`: `CashIn`
            -   `Value`: `row("CashIn").ToString()`
        ii. **Add another `Assign` activity.**
            -   `DisplayName`: "Extract OnUsCheck"
            -   `To`: `OnUsCheck`
            -   `Value`: `row("OnUsCheck").ToString()`
        iii. **Add a third `Assign` activity.**
            -   `DisplayName`: "Extract NotOnUsCheck"
            -   `To`: `NotOnUsCheck`
            -   `Value`: `row("NotOnUsCheck").ToString()`
        iv. **Add an `Add Queue Item` activity.**
            -   `DisplayName`: "Add Item to Orchestrator Queue"
            -   `QueueName` property: `in_Config("OrchestratorQueueName").ToString()`
            -   `Reference` property: `row("TransactionID").ToString()` (assuming a unique ID column exists)
            -   In the `ItemInformation` collection property, add the following key-value pairs:
                -   Key: `"CashIn"`, Value: `CashIn`
                -   Key: `"OnUsCheck"`, Value: `OnUsCheck`
                -   Key: `"NotOnUsCheck"`, Value: `NotOnUsCheck`

    c. **Inside the `Catches` block of the `Try Catch` activity:**
        i. **Add a new `Catch` for `System.Exception`** with a variable named `exception`.
        ii. Inside this catch, add a **`Log Message` activity.**
            -   `DisplayName`: "Log Failed Item"
            -   `LogLevel`: `Error`
            -   `Message`: `"Failed to add item to queue. Error: " + exception.Message`

4.  **After the `For Each Row` loop, add a final `Assign` activity.**
    -   `DisplayName`: "Set Upload as Complete"
    -   `To`: `UploadComplete`
    -   `Value`: `True`
