# Azure DevOps Team History Report

## Description

This is a prompt that can be run ad-hoc in Claude Code. It takes a list of comma separated Ids, pulls all ticket info and ticket history for all of those AzDO ticket Ids, and creates reports for that data.  

It then filters out all of that history to only show/process historical changes made in the last X number of days by users in the User Names table at the top of the prompt. 

Taking this filtered data, it generates two reports in the path provided to the prompt. I have this pointed to my local Obsidian vault's directory so that I can easily keep track of them. 

The first report is a table just of each and every change made on the tickets by users in the User Names table. It includes Name of the user that did the change, the date that change was made, if the change was creating a ticket, if that change was making a comment, or if that change was something else changing a field. Rows where all three of the last columns are "false" is some other, unmeaningful change such as adding a link to the ticket.

The table generated is very easy, in Obsidian, to copy and paste into Excel where you can then add additional cells to help provide totals to see your teams change rate. 

The second report breaks down who made how many changes on each ticket. For every ticket where the filtered changes apply, the users who made those changes are listed in a table. That table shows whether the user created the ticket, how many comments they made on that ticket, and how many "other" field changes they made on that ticket. 

At the bottom of the second report are all the tickets where, after filtering, no changes were found.

## Edits Needed

The included `prompt.md` file has been scrubbed of identifying information. You should edit it before pasting it into Claude Code with the below changes. 

| Word | Change Needed |
|----------|----------|
| `{user_name}` | The name of the user('s) that you are running this report for |
| `{ids}` | A comma separated list of AzDO Ticket Ids you want to check |
| `{organization}` | The name of the Azure DevOps organization you are connecting to |
| `{project}` | The name of the Azure DevOps project you are connecting to |
| `YOUR_BASE64_PAT` | Your Azure DevOps Personal Access token, converted to Base64. Can be done by opening a browser and running `btoa(':YOURPAT')` in a browser console. |
| `{min_date}` | The number of days you want to see history for prior to today |
| `{full_report_path}` | The path with file name of where you want the full history report saved |
| `{ticket_report_path}` | The path with file name of where you want the ticket history report saved |

## Requirements 

This prompt requires: 

- Ability to run a Local Routine in Claude Desktop 
- Azure DevOps Personal Access Token with Read permissions on Tickets
- A local Obsidian Vault

## Notes about Azure DevOps Users and Tickets

- `User Names`: The `User Names` table at the top of this prompt can contain any number of users you want to include in these reports. However, the name much match exactly with that user's name in Azure DevOps
- Ticket Ids: I personally pull these ticket Ids from an Azure DevOps query. I run a query that contains all tickets under an Area Path, where Work Item Type is in "Product Backlog Item,Bug,Feature", and Changed Date >= @StartOfDay('-Yd') where in this case Y is `{min_date} + 1`. You can then export the results to a `.csv` file, copy the Ids of the tickets into a text editor, and manually add commas. Using vs-code and ALT+SHIFT+Click is super helpful here. 
- Changes with a date of "Current/Open" are typically the most recent change made on the ticket. For some reason, the AzDO Updates API returns some dates as `9999-01-01T00:00:00Z`. As far as I can tell, that just means it's the "Current" change on that ticket.


## Additional Note

This prompt will need to be babysat in order to run without having to manually give Claude permissions several times. 

In order to get around that, you can set your Claude Code settings on this prompt to "Bypass permissions".

**Only do this if you are aware of the consequences and this is your local machine that only you have access to**