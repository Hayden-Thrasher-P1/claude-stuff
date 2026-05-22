# Azure DevOps Ticket Mentions Triage

## Description

This routine runs daily locally at 12:45 PM. It checks the Outlook email account of the user setup, and looks for emails in the `AzDO Mentions` email folder. 

It groups all mentions by Type (Feature, PBI, Bug) and then Ticket ID. It then uses the AzDO API to pull Ticket info and comment treads. 

It uses context of each ticket to see if action is needed by the user. 

If action is required, it is treated as "Action Needed". If No action is required, it is categorized as "No Action Needed". 

Claude then provides a one sentence update on what action needs to be taken by the user. 

A file in the user's local Obsidian vault is then updated with all items found, and recommendations of actions needed taken. 

## Edits Needed

The included `routine-prompt.md` file has been scrubbed of identifying information. You should edit it before creating your routine with the below changes. 

| Word | Change Needed |
|----------|----------|
| `user`  | The name of the person this is being run for | 
| `user@email.com`  | The email address of the Outlook account this is being run for | 
| `{organization}` | The name of the Azure DevOps organization you are connecting to |
| `{project}` | The name of the Azure DevOps project you are connecting to |
|  `YOUR_BASE64_PAT` | Your Azure DevOps Personal Access token, converted to Base64. Can be done by opening a browser and running `btoa(':YOURPAT')` in the console. 

## Requirements 

This routine requires: 

- Ability to run a Local Routine in Claude Desktop 
- A Microsoft365 account, and setting up the M365 connector in Claude Desktop
- Azure DevOps Personal Access Token
- A local Obsidian Vault