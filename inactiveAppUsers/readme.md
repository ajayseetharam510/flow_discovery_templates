# Workflows Template: Identify Inactive App Users to save per-user app license costs

## Overview


In many organizations there are specific applications where the per-user license cost is high. Very often, users are provided access to these  applications when they need to complete a task. They later continue to consume a seat license even though they do not access the application any  longer. With this sample workflow flopack we are able to monitor access to these specific applications and take action, as needed. The value for customers is significant application license cost savings. 

## Before you get Started/Pre-requisites: 

Before you get started, you will need:
- Access to an Okta tenant with Okta Workflows enabled for your org 
- Make sure the `` okta.logs.read ``  API scope is enabled for the ``Okta Workflows OAuth`` app. If you grant it, then you must reauthorize the Okta Workflows Okta Connection for it to take effect.
- A test application configured in Okta that you can access - for example, a SAML test app (friendly name will be referred to as ``My  High Cost App``). Note the application Instance ID (appInstanceId). In the instructions below the appInstanceId is ``0oa1msww4rqy48tjr1d7`` as noted in the Okta admin console app configuration URL - your value will be different.
  - Example: If the url when you access the app configurations in the Okta Admin console is `https://...okta.com/admin/app/..._samltestapp_1/instance/0oa1msww4rqy48tjr1d7/#tab-general` your appInstanceId is `0oa1msww4rqy48tjr1d7`
- Sample users with known credentials that are assigned to the test app.
- The set of flows in this template are organized as below: 
  - Initial set up (these flows can be turned off after initialization): 
      - `[0.0] Use for Initial set up of app users in tracker tab` 
      - `[0.1] Load initial user to tracker table` 
  - Ongoing additions/removals of users assigned to the application from the `Track User Access` Table. These api endpoints are registered as Event Hooks.  
      - `[0.3] On App assignment`
      - `[0.4] On App unassignment`
  - On a schedule, query syslog to check for `user.authentication.sso` events for the specific applications configured in the `Configuration Data` table and for those users update the `LastAccessDate` in the `Track User Access` table.
      - `[1.0] - QuerySysLogs for App Authentications`
      - `[1.1] - Query syslog for specific appInstanceId`
      - `[1.2] - Record User access to app to track history`
  - Periodically query the `Track User Access` Table to find users that have not accessed the app for the past X days (i.e. value based on `Configuration Data`) and toggle the value in the `InactiveFlagToReviewAssignment` column from `false` to `true`
      - `[2.0] Check for Inactive App Users`
      - `[2.1] Find inactive users for specific appInstance`
      - `[2.2] Mark row as Inactive`
  - Deliver the list of inactive users via email
      - `[3.0] Deliver list of inactive users`
      - `[3.1] Process specific appinstanceId`
      - `[3.2] Create a list of users for specific appInstanceId`


## Setup Steps to configure the Okta Workflows: 

- Perform the basic configurations in the flows as outlined below
  - `[0.0] Use for Initial set up of app users in tracker tab` - set the Okta connection
  - `[0.0] Use for Initial set up of app users in tracker table` - set the `List Users Assigned to App` card Options to reference the test app and save.
  - `[0.1] Load initial user to tracker table` - populate the Assign card with the appInstanceId for your app and save.

  - `[1.1] - Query syslog for specific appInstanceId` - set the Okta connection
  - `[3.1] Process specific appinstanceId` - set the email connection. If using a different email provider then please replace that connection and mapping, as appropriate. 
 
- Populate configuration values in the Okta Workflows table `Configuration Data`
  - Create a row in the `Configuration Data` table with your values. Note: enter the values carefully - appInstanceId, Status as  New, days as a number and your email are all key for this to work. 
  
  
      | **Column Name** | **Value that should be configured**  | 
      |:----------|:----------|
      | appInstanceId   | The test application Okta appInstanceId | 
      | Days to mark as inactive  | 30   | 
      | Status  | New    | 
      | App Name  | My High Cost App   | 
      | Email to notify  | Enter your email address  | 
 - Turn on all the flows
 - Manually run the flow `[0.0] Use for Initial set up of app users in tracker table`. 
   - Verify: You should now see all users that are currently assigned to the test app listed in the `Track User Access` table.
  
- Now turn off the following two flows 
  - `[0.0] Use for Initial set up of app users in tracker table`
  - `[0.1] Load initial user to tracker table`
  - Note: In the future when you set up an additional `appInstanceId` you will repeat the above steps of adding a row to the `Configuration Data` and populating the `Track User Access` table with the users assigned to the Okta application.  
  
- Create & Activate **one** Event Hook in the Okta Admin Console for the **two events** - `User assigned to app` and `User unassigned from app`, with the same filter expression for **BOTH** events similar to the sample Expression Language provided below and the webhook endpoint value from the `[0.3] On App assignment` flow. 
  - Example Expression Language (for multiple applications): `event.target.?[type eq 'AppInstance' && id eq '0oa1msww4rqy48tjr1d7'].size() >0 OR event.target.?[type eq 'AppInstance' && id eq '0oa4cne1xokgO7pu21d7'].size() >0`
  
- Manually change the `Configuration Data` table entry for the appInstanceId from “New” to “Ready” after you have completed this initial set up for  the application. This will have the `appInstanceId` get processed from now on.
 
## Testing this flow
- Access the app as your test user by logging in from a separate browser window. 
- Execute the following flow manually (it is set up to run on a schedule)
  -  `[1.0] - QuerySysLogs for  App Authentications` 
- Verification: Look at the table `Track User Access` - the `LastAccessDate` value is updated for the test user. The `InactiveFlagToReviewAssignment` is set to  false
- To simulate a user that has not accessed the app for a long time manually change the `LastAccessDate` column in the `Track User Access` table  to a value at least 31 days prior to current date.
- Execute the following flow manually (it is set up to run on a schedule)
  - `[2.0] Check for Inactive App Users flow` 
- Verification: Look at the table `Track User Access` - the `InactiveFlagToReviewAssignment` is set to true for the simulated user
- Execute the following flow manually (it is set up to run on a schedule)
  - `[3.0] Deliver list of inactive users`
- Verification: An email is delivered to the configured email address with the simulated user listed


## Limitations & Known Issues

- Keep in mind the [Okta Workflows System Limits](https://help.okta.com/en/prod/Content/Topics/Workflows/workflows-system-limits.htm) to limit the entries in the Okta table and the Okta API rate limits. 
- Note: This template will work only for apps that generate app authentication events in okta syslog that match
  
    `event type` = `user.authentication.sso` 

  Therefore, it will not track Okta bookmark app access, for example. 
- Consider app session times and the authentication policy - the design is dependent on okta app authentication events. 
- The template is useful to identity inactive app users when the number of inactive days is set to durations of multiple weeks (example, 30 days) rather than being accurate in the minutes range. Syslog events are queried every 30 minutes but not processed in any specific time order.   
