# Transfer user's Google Drive files and folders to another user in GSuite 

## Overview


In many organizations that use Google Drive, there is a requirement to transfer a user's GDrive contents to another user. May be you have users that need to be deactivated - you can transfer the files from the user's GDrive to the Manager and delete the user.

## Before you get Started/Pre-requisites: 

Before you get started, you will need:
- Access to an Okta tenant with Okta Workflows enabled for your org 
- Create a free GSuite Trial account from [here](https://gsuite.google.com/) - click on "Get Started". You will need a credit card to sign up for a free 14 day trial. 

## Setup Steps to configure the Okta Workflows: 
- In Okta Workflows, Create a Google Drive Connection Setting. To set up a new configuration in Google Drive:
    - Enter in an Account Nickname. This should be unique. If you are connecting multiple Google Drive accounts, then you’ll be able to tell them apart.
    - Click **Create** to finish this configuration.
- Configure Okta Workflows flows to use the connection setting you just created and the "GSuite Drive Transfer" Okta Workflows table:
    - In "[1.0] Start Transfer", choose the Google Drive connection setting for the "Create Transfer Request" card. Turn the flow On.
    - In "[1.1] Read Specific Transfer Status" choose the Google Drive connection setting for the "Read Transfer Request" card. Turn the flow On.
    - In "[1.0] Read Transfer Status as Scheduled Process" choose Options -  "GSuite Drive Transfer" Okta Workflows table with "All Matching Rows". Turn the flow On. 

## Testing this flow
-As Google Administrator access the [Google Admin console](https://admin.google.com) 
- Create two users - for example, `Luke Skywalker lukeskywalker@<your_domain.com>` and `Obiwan Kanobi obiwankanobi@<your_domain.com>` with known passwords for convenience during your test. (Replace <your_domain.com> with your domain. 
- Assign "DocsMailDrive" to users from all organizational units. 
- As `lukeskywalker@<your_domain.com>` access [Google Drive](https://drive.google.com)
- Create a test document in Google Drive account of `Luke Skywalker` - we will transfer this document to `Obiwan Kanobi` 
- On Workflows console click on play i.e. Run Flow icon for  "[1.0] Start Transfer" flow  to test it
- Enter "from" and "to" parameters as:
`lukeskywalker@<your_domain.com>` and
`obiwankanobi@<your_domain.com>` respectively
- Observe the entry in the "GSuite Drive Transfer" Okta Workflows table with the "G Suite Tranfer Request ID", "From" and "To" fields populated
- On Workflows console click on play i.e. Run Flow icon for  "[1.0] Read Transfer Status as Scheduled  Process" flow  to test it
- Re-run (if needed) until the entry in the  "GSuite Drive Transfer"  table is deleted
- As `lukeskywalker@<your_domain.com>` access [Google Drive](https://drive.google.com) Check that the test document is not in Luke Skywalker's GDrive account. 
- As `obiwankanobi@<your_domain.com>` access [Google Drive](https://drive.google.com) Check that the test document is in Obiwan Kanobi's GDrive account.  

## Limitations & Known Issues

- Keep in mind the [Okta Workflows System Limits](https://help.okta.com/en/prod/Content/Topics/Workflows/workflows-system-limits.htm) to limit the entries in the "GSuite Drive Transfer Tracker" Okta table. 
- The "GSuite Drive Transfer Tracker" Okta table is not intended to be an audit log.
