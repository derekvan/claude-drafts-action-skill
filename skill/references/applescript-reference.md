# AppleScript Reference for Drafts

**Platform:** macOS only

This reference covers AppleScript integration with Drafts, including both running AppleScript from within Drafts actions and controlling Drafts from external AppleScript.

## Using AppleScript from Drafts

### The "Run AppleScript" Action Step

The "Run AppleScript" action step allows you to execute AppleScript code directly within a Drafts action, with access to the current draft's properties.

#### Structure

Each AppleScript should contain an `execute` subroutine that takes a single parameter containing draft information:

```applescript
on execute(draft)
    -- Your AppleScript code here
    return "Optional return value"
end execute
```

#### Draft Record Properties

The `draft` parameter passed to your AppleScript contains these properties:

**Core Properties:**
- **uuid** *(text)*: Unique identifier
- **content** *(text)*: Full draft text
- **title** *(text, readonly)*: First line of draft
- **tags** *(list of text)*: Array of tag names
- **flagged** *(boolean)*: Flag status

**Metadata:**
- **folderName** *(text)*: inbox, archive, trash, or flagged
- **languageGrammar** *(text)*: Syntax identifier (e.g., "Markdown", "JavaScript")
- **permalink** *(string)*: Draft permalink URL

**Timestamps:**
- **createdAt** *(date)*: Creation date/time
- **modifiedAt** *(date)*: Last modification date/time

**Location Data:**
- **createdLatitude** *(number)*: Latitude where draft was created
- **createdLongitude** *(number)*: Longitude where draft was created
- **modifiedLatitude** *(number)*: Latitude where last modified
- **modifiedLongitude** *(number)*: Longitude where last modified

#### Example: Debug Draft Properties

```applescript
on execute(draft)
    set theText to ""
    
    set theText to theText & "UUID: " & uuid of draft & return
    set theText to theText & "TITLE: " & title of draft & return
    set theText to theText & "CREATED: " & createdAt of draft & return
    set theText to theText & "MODIFIED: " & modifiedAt of draft & return
    set theText to theText & "FOLDER: " & folderName of draft & return
    set theText to theText & "FLAGGED: " & flagged of draft & return
    set theText to theText & "SYNTAX: " & languageGrammar of draft & return
    set theText to theText & "PERMALINK: " & permalink of draft & return
    
    display dialog theText
    
    return "Properties displayed"
end execute
```

#### Return Values

If your AppleScript returns a result, it will be available to subsequent Script action steps via `context.appleScriptResponses` array. Most basic data types (strings, numbers, booleans, lists, records) are converted to JavaScript values.

Example accessing returned value in JavaScript:

```javascript
// After a Run AppleScript step that returns a value
let result = context.appleScriptResponses[0];
console.log("AppleScript returned: " + result);
```

#### Development Workflow

**Recommended approach:**
1. Write and test scripts in Apple's Script Editor application
2. Use a mock `execute` call for testing:

```applescript
on execute(draft)
    -- Your script logic
end execute

-- Mock for testing in Script Editor
execute({title:"Test Title", content:"Test Content", uuid:"123", tags:{"test"}, flagged:false, folderName:"inbox", languageGrammar:"Markdown", createdAt:current date, modifiedAt:current date, createdLatitude:0.0, createdLongitude:0.0, modifiedLatitude:0.0, modifiedLongitude:0.0, permalink:"https://example.com"})
```

3. Copy final working script into Drafts action editor

### The AppleScript Script Object

For more advanced and customizable AppleScript execution, use the `AppleScript` JavaScript object in a Script action step.

#### Basic Usage

```javascript
// Define your AppleScript
let method = "execute";
let script = `on execute(bodyHTML)
    tell application "Safari"
        activate
    end tell
    return "Success!"
end execute`;

// Prepare arguments
let html = draft.processTemplate("%%[[draft]]%%");

// Execute
let runner = AppleScript.create(script);
if (runner.execute(method, [html])) {
    // Success - access return value
    alert(runner.lastResult);
} else {
    // Error occurred
    alert(runner.lastError);
}
```

#### Key Methods

**`AppleScript.create(script)`**
- Creates an AppleScript runner with the provided script text
- Returns: AppleScript object

**`runner.execute(methodName, arguments)`**
- Executes the named subroutine with provided arguments
- `methodName`: String name of subroutine to call
- `arguments`: Array of values to pass (optional)
- Returns: Boolean (true if successful)

**Properties:**
- `runner.lastResult`: Value returned by the AppleScript
- `runner.lastError`: Error message if execution failed

#### Example: Process Draft and Return Result

```javascript
let script = `on processText(inputText)
    -- Transform text (e.g., uppercase)
    set uppercaseText to do shell script "echo " & quoted form of inputText & " | tr '[:lower:]' '[:upper:]'"
    return uppercaseText
end processText`;

let runner = AppleScript.create(script);
if (runner.execute("processText", [draft.content])) {
    draft.content = runner.lastResult;
    draft.update();
    app.displaySuccessMessage("Text transformed");
} else {
    context.fail("AppleScript error: " + runner.lastError);
}
```

#### Error Handling

Always check the return value and handle errors:

```javascript
let runner = AppleScript.create(script);
if (!runner.execute("myMethod", [arg1, arg2])) {
    console.log("AppleScript failed: " + runner.lastError);
    app.displayErrorMessage("Script execution failed");
    context.fail();
}
```

## Using Drafts from AppleScript

Drafts can be controlled from external AppleScript, allowing integration with other Mac automation workflows.

### Core Capabilities

**Drafts:**
- Create new drafts
- Update content and properties
- Query and filter drafts
- Get current draft from editor

**Workspaces:**
- Query available workspaces
- Load drafts from specific workspaces
- Get current workspace

**Actions:**
- Query available actions
- Run actions on drafts or text

### Creating Drafts

#### Basic Creation

```applescript
tell application "Drafts"
    make new draft with properties {content:"My Draft Content"}
end tell
```

#### Creation with Properties

```applescript
tell application "Drafts"
    make new draft with properties {¬
        content:"My Draft Content", ¬
        flagged:false, ¬
        tags:{"blue", "green"}}
end tell
```

#### Creation with Full Metadata

```applescript
tell application "Drafts"
    make new draft with properties {¬
        content:"Project Notes", ¬
        folder:inbox, ¬
        flagged:true, ¬
        tags:{"work", "urgent"}, ¬
        languageGrammar:"Markdown"}
end tell
```

### Querying Drafts

#### By Tags

```applescript
tell application "Drafts"
    set myDrafts to every draft whose tags contains "personal"
end tell
```

#### By Folder and Flag Status

```applescript
tell application "Drafts"
    set myDrafts to every draft whose folder is equal to inbox and flagged is equal to true
end tell
```

#### By Content

```applescript
tell application "Drafts"
    set myDrafts to every draft whose content contains "Hello"
end tell
```

#### By UUID

```applescript
tell application "Drafts"
    set myDraft to draft id "4A376C15-73B4-48DE-9C7B-1BD5FF9C65D9"
end tell
```

#### Current Draft

```applescript
tell application "Drafts"
    set myDraft to current draft
end tell
```

### Getting Draft Properties

```applescript
tell application "Drafts"
    set myDraft to current draft
    
    -- Get properties
    set myContent to content of myDraft
    set myTitle to title of myDraft
    set myTags to tags of myDraft
    set myFolder to folder of myDraft
    set isFlagged to flagged of myDraft
    set myUUID to uuid of myDraft
    set myCreated to createdAt of myDraft
    set myModified to modifiedAt of myDraft
end tell
```

### Updating Drafts

#### Update Content

```applescript
tell application "Drafts"
    set myDraft to draft id "4A376C15-73B4-48DE-9C7B-1BD5FF9C65D9"
    set content of myDraft to "New Content"
end tell
```

#### Update Folder

```applescript
tell application "Drafts"
    set myDraft to current draft
    set folder of myDraft to archive
end tell
```

#### Update Tags

```applescript
tell application "Drafts"
    set myDraft to current draft
    set tags of myDraft to {"red", "purple"}
end tell
```

#### Update Multiple Properties

```applescript
tell application "Drafts"
    set myDraft to draft id "4A376C15-73B4-48DE-9C7B-1BD5FF9C65D9"
    set content of myDraft to "Updated Content"
    set folder of myDraft to archive
    set tags of myDraft to {"red", "purple"}
    set flagged of myDraft to true
end tell
```

#### Append to Draft

```applescript
tell application "Drafts"
    set myDraft to current draft
    set content of myDraft to (content of myDraft) & return & return & "Appended text"
end tell
```

### Working with Workspaces

#### List All Workspaces

```applescript
tell application "Drafts"
    get workspaces
end tell
```

#### Get Specific Workspace

```applescript
tell application "Drafts"
    set myWorkspace to workspace "Work"
end tell
```

#### Get Current Workspace

```applescript
tell application "Drafts"
    set myWorkspace to current workspace
end tell
```

#### Query Drafts in Workspace

```applescript
tell application "Drafts"
    -- Drafts in specific workspace
    every draft of workspace "Work" whose folder is inbox
    
    -- Drafts in current workspace
    set myDrafts to every draft of current workspace whose folder is inbox
end tell
```

#### Filter by Multiple Criteria

```applescript
tell application "Drafts"
    set myDrafts to every draft of workspace "Personal" ¬
        whose folder is inbox ¬
        and flagged is true ¬
        and tags contains "important"
end tell
```

### Running Actions

#### Get Action

```applescript
tell application "Drafts"
    set myAction to action "Copy"
end tell
```

#### Run Action on Draft

```applescript
tell application "Drafts"
    set myAction to action "Copy"
    set myDraft to draft id "4A376C15-73B4-48DE-9C7B-1BD5FF9C65D9"
    perform action myAction on draft myDraft
end tell
```

#### Run Action on Current Draft

```applescript
tell application "Drafts"
    set myAction to action "Tweet"
    set myDraft to current draft
    perform action myAction on draft myDraft
end tell
```

#### Run Action on Text

```applescript
tell application "Drafts"
    set myAction to action "Process Text"
    perform action myAction on text "Some text to process"
end tell
```

### Complete Workflow Examples

#### Process Inbox Items

```applescript
tell application "Drafts"
    -- Get all flagged inbox drafts
    set inboxDrafts to every draft whose folder is inbox and flagged is true
    
    repeat with aDraft in inboxDrafts
        -- Add processed tag
        set currentTags to tags of aDraft
        set tags of aDraft to currentTags & {"processed"}
        
        -- Archive the draft
        set folder of aDraft to archive
        
        -- Unflag
        set flagged of aDraft to false
    end repeat
    
    display notification "Processed " & (count of inboxDrafts) & " drafts"
end tell
```

#### Create Daily Note

```applescript
tell application "Drafts"
    set today to current date
    set dateString to (year of today as string) & "-" & ¬
        text -2 thru -1 of ("0" & (month of today as integer)) & "-" & ¬
        text -2 thru -1 of ("0" & (day of today as integer))
    
    set noteContent to "# Daily Note: " & dateString & return & return & "## Tasks" & return & return & "## Notes"
    
    make new draft with properties {¬
        content:noteContent, ¬
        tags:{"daily"}, ¬
        flagged:true}
end tell
```

#### Batch Tag Update

```applescript
tell application "Drafts"
    -- Find drafts with old tag
    set oldTagDrafts to every draft whose tags contains "todo"
    
    repeat with aDraft in oldTagDrafts
        set currentTags to tags of aDraft
        
        -- Remove old tag and add new tag
        set newTags to {}
        repeat with aTag in currentTags
            if aTag is not "todo" then
                set end of newTags to aTag
            end if
        end repeat
        set end of newTags to "task"
        
        set tags of aDraft to newTags
    end repeat
end tell
```

#### Export Drafts to Files

```applescript
tell application "Drafts"
    set myDrafts to every draft of workspace "Writing" whose folder is archive
    
    repeat with aDraft in myDrafts
        set fileName to title of aDraft & ".md"
        set filePath to (path to documents folder as string) & fileName
        set fileContent to content of aDraft
        
        try
            set fileRef to open for access file filePath with write permission
            set eof of fileRef to 0
            write fileContent to fileRef
            close access fileRef
        on error
            close access file filePath
        end try
    end repeat
end tell
```

## Folder Constants

When setting or querying the `folder` property, use these constants:

- `inbox` - Inbox folder
- `archive` - Archive folder
- `trash` - Trash folder
- `flagged` - Flagged folder (virtual folder showing flagged drafts)

Example:
```applescript
set folder of myDraft to archive
set inboxDrafts to every draft whose folder is inbox
```

## Tips and Best Practices

### Development

1. **Use Script Editor**: Develop and test in Apple's Script Editor app before copying to Drafts
2. **Add mock data**: Include test `execute` calls for debugging
3. **Check app dictionary**: Use Script Editor's "Open Dictionary" to view Drafts' full AppleScript dictionary

### Error Handling

1. **Wrap in try blocks**: Use AppleScript's error handling for robustness

```applescript
on execute(draft)
    try
        -- Your code
        return "Success"
    on error errMsg
        display dialog "Error: " & errMsg
        return "Failed"
    end try
end execute
```

2. **Validate inputs**: Check draft properties before using them

```applescript
on execute(draft)
    if length of content of draft is 0 then
        return "Draft is empty"
    end if
    -- Process non-empty draft
end execute
```

### Performance

1. **Batch operations**: When querying multiple drafts, do all queries in one `tell` block
2. **Filter early**: Use specific queries to limit results rather than filtering in AppleScript
3. **Minimize UI updates**: Avoid excessive `display dialog` calls in loops

### Integration

1. **Combine with JavaScript**: Use AppleScript object in JavaScript for complex workflows
2. **Use with Shortcuts**: Integrate Drafts AppleScript in Shortcuts.app automation
3. **Workflow automation**: Trigger Drafts actions from other Mac automation tools (Keyboard Maestro, Alfred, etc.)

## Resources

- **Script Editor**: macOS built-in app for writing and testing AppleScript
- **Drafts Scripting Dictionary**: View in Script Editor via File → Open Dictionary → Drafts
- **Example Actions**: Install [Examples (Mac): AppleScript & Shell Script](https://actions.getdrafts.com/g/16C) action group
- **Forums**: Get help at [forums.getdrafts.com](https://forums.getdrafts.com)
