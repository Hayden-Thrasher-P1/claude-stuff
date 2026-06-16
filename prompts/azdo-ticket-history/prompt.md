
Run this routine with --dangerously-skip-permissions enabled.

You are an automated assistant.

Your job is to run the AzDO Ticket History Report. Do the following steps in order, completing each fully before moving to the next. 

The below table is the Users that you will be looking for. I will refer to this table as "User Names" in this document. 

| User Names |
|------------|
| {user_name} |

Complete all steps using a bash shell script. Write the script to /tmp/azdo_history_run.sh, then execute it. Do not use Python or any other language. In the bash script, fetch all work item details in parallel using background processes (&) and wait, then do the same for history. Do not fetch tickets sequentially. Run no more than 20 parallel requests at a time using a concurrency limiter pattern (e.g. process slots with wait). If the script already exists at /tmp/azdo_history_run.sh, overwrite it completely before running.

## Step 1 — Get all ticket IDs
The below is a comma separated list of ticket Ids. These are the ids of tickets in Azure DevOps. Make sure not to save spaces or the commas in the Ticket Ids you will use.

Ids: {ids}

Make a list of all the unique ticket Ids. 

## Step 2 - Fetch the ticket history

For each unique ticket ID, make two REST API calls one for work item details and one for work item history. Run these in parallel first getting details for all the tickets and then getting history for all tickets.

Do not try to open this in a web browser, by using WebFet, nor by using the Azure DevOps MCP connector. Get this information using direct REST API calls w/ Basic auth token as specified below. 

Get work item details:
GET https://dev.azure.com/{organization}/{project}/_apis/wit/workitems/{id}?$expand=all&api-version=7.1
Header — Authorization: Basic YOUR_BASE64_PAT
Header — Content-Type: application/json

Get work item history:
GET https://dev.azure.com/{organization}/{project}/_apis/wit/workItems/{id}/updates?api-version=7.1
Header — Authorization: Basic YOUR_BASE64_PAT
Header — Content-Type: application/json

From the work item extract: title, assigned to. 

From the history of each ticket follow the below: 

Each history response is JSON. That JSON contains a property "value" which is an array of all the history changes for the ticket. 

For each Ticket, save a list of history changes made and extract from each change the following. Use the table below. In the first column is the "Readable" property name that you should save from the change. The second column is the full JSON property name of the cooresponding value. I am using `i` in arrays to denote that this should be available in every entry of a given array. 

| Readable Name | JSON Property Name |
|---------------|--------------------|
| Changed By | value[i].revisedBy.displayName |
| Changed Date | value[i].revisedDate |

If the Changed date is something like "9999-01-01T00:00:00Z" where the date is far in the future, Change the date to "Current/Open". Otherwise keep the date the same.

Additionally, the following rules apply to the history data you will extract: 

- There should be an extracted readable property name "Ticket Created". If the value[i].id of the history is one and/or the valude[i].rev of the history is one, extract "Ticket Created" as "true". Otherwise, "Ticket Created" for that property on the history should be "false". 

- There should be an extracted readable property name "Fields Changed". If "Ticket Created" is "false" for a history AND there is an object on the history item called "fields", extract "Fields Changed" with the property value "true". Otherwise, "Fields Changed" for that property on the history should be "false". The JSON property name will be like value[i].fields

- There should be an extracted readable property name "Comment Made". If "Ticket Created" is "false" for a history AND if the "fields" object exists, AND there is another object in the fields named "System.CommentCount", extract "Comment Made" with the property value "true". Otherwise, "Comment Made" should be "false". The JSON property name will be like value[i].fields.System.CommentCount

If any call fails for a ticket, note it and continue - do not stop the whole run. Track which ones fail. 

## Step 3 — Filter the history returned
For each ticket, filter out any History objects that do not match ALL of the below filters: 

- Changed By name is in "User Names" table at the top of this doc.
- Changed Date is >= {min_date} days prior to today's date OR "Current/Open"

For tickets that have history, save them in state "Has History". 

For tickets with no history, save them in state "Has No History". 

## Step 4 - Add additional parameters to filtered data
After performing the above filters, add data properties to each ticket. The table below provides the Readable Name in the first column, and how that property should be calculated in the second column. 

Each of these properties needs to be saved and grouped PER Changed By name. So there will be a User1_Created?, User1_Comment Count, User1_Change Count, User2_Created?, User2_Comment Count, etc etc. 

| Readable Name | How to Calculate |
|---------------|------------------|
| Created? | If any history for this ticket has "Ticket Created" = "true" for this user, this value is "Yes". Otherwise this value is "No" |
| Comment Count | Total number of histories for this ticket where "Comment Made" = "true" for this user|
| Change Count | Total number of histories for this ticket where "Comment Made" = "false" and "Fields Changed" = "true" for this user |

Finally, calculate full totals for all tickets. The table below provides the Readable Name in the first column of these full totals, and how that property should be calculated in the second column. 

| Readable Name | How to Calculate |
|---------------|------------------|
| Created_Count | Total number of tickets where "Created?" = "Yes" |
| All_Comment_Count | Sum of all "Comment Count" properties for all tickets |
| All_Change_Count | Sum of all "Change Count" properties for all tickets |

## Step 5 — Create First Markdown Report
Write the first completed report to this exact path, overwriting the file completely each run:
{full_report_path}.md

Use this exact structure:

# AzDO Full History Report
**Generated:** {Day, Month DD YYYY at h:mm AM/PM}

---

### All History

| Changed By | Changed Date | Ticket Created | Fields Changed | Comment Made |
|------------|--------------|----------------|----------------|--------------|
| {changed by} | {date} | true/false | true/false | true/false |

Other Rules for the report:
- Date format: MMM DD h:mm AM/PM (e.g. May 22 3:42 PM)


## Step 6 - Create Second Markdown Report
Write the first completed report to this exact path, overwriting the file completely each run:
{ticket_report_path}.md

Use this exact structure:

# AzDO History Report
**Generated:** {Day, Month DD YYYY at h:mm AM/PM}

---

### Has History 

- [ ] **{ID} — {Title}** [Open in AzDO](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID})
> {Assigned To}

| Changed By | Created? | Change Count | Comment Count | 
|------------|----------|--------------|---------------|
| {user} | {user_Created?} | {user_Change Count} | {user_Comment Count} |

### Totals

**Tickets Created**: {Created_Count}
**Total Comment Count**: {All_Comment_Count}
**Total Change Count**: {All_Change_Count}

### Has No History

- [] **{ID} - {Title}** [Open in AzDO](https://dev.azure.com/{organization}/{project}/_workitems/edit/{ID})
> {Assigned To}

Other Rules for the report:
- Date format: MMM DD h:mm AM/PM (e.g. May 22 3:42 PM)
- If a ticket's AzDO fetch failed, add a note in italics under the title: *⚠️ Could not fetch ticket details — review manually*. These should be in the "Needs Action" section of the report.

At the end of this, remove the script you created in `tmp/azdo_history_run.sh`.