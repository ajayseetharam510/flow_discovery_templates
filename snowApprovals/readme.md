# Approve Okta Group access in ServiceNow 

## Overview


In many organizations that use ServiceNow, a subset of access may require approvals. May be you have users that are provisioned birthright access when created but a specific group access needs to be approved before being provisioned. For these usecases you can get approvals using ServiceNow. 

## Before you get Started/Pre-requisites: 

Before you get started, you will need:
- Access to an Okta tenant with Okta Workflows enabled for your org 
- A staged user with Department set to "Corellia". You will activate this user to trigger a workflow approval in ServiceNow
- Create a free Personal Development Instance of the latest ServiceNow release at <https://developer.servicenow.com> 
- Set up ServiceNow Connection setting in Okta Workflows
	- [x] In the Username and Password fields, type your **admin** ServiceNow login credentials. In the Instance field, enter your ServiceNow subdomain value.You can find the Service Now subdomain value in the instance’s URL. For example in *https://dev57240.service-now.com/* the instance name will be *dev57240*.
- Make sure the Okta workflows reference the ServiceNow Connection setting, Okta org setting and point to the Okta table created
- Create a target group to be assigned in Okta and note the groupId (easy to get this from the URL of the group from the Okta Admin Console)
- Update the Process Resolution flow to put in the groupId as noted in the flow
- Turn on the flows
- Save the **Okta Workflows** **Invoke URL** api endpoint of the **Process Resolution** workflow since you will need it when you configure the **ServiceNow** **RESTMessage**

## Setup Steps to configure the Personal Development Instance of ServiceNow: 

### Incidents Form Design


1.  Type **Incidents** in the Filter Navigator at the top left and
    adjust filter criteria to see a list of incidents. This is sample
    data that is pre-populated in the ServiceNow instance.

2.  Click on **Incidents** under the Self-Service category.

3.  Click on any Incident.

>
> We will be adding some fields to this type of incident. The fields can
> be added by clicking on the **hamburger icon** at the top left.


4.  Click on **Configure** and then click on **Form Design.**

5.  Underneath the Fields menu, scroll down to find **Resolution
    notes**. Drag the tile over to the Incident and drop it underneath
    **Watch List**.


6.  Do the same for **Resolution code**.

7.  Scroll up to find the **Approval** field and drag it underneath
    **On** **hold reason**.

8.  Click **Save** at the upper right of the web page.

9. Close the "form design browser" tab.


### Create a REST Message

1.  In the Filter Navigator type **REST Message** and click REST Message
    under Outbound. This is where we will be configuring the call back
    to Okta workflows.


2.  Click on **New** to create a REST Message.

3.  In the **Name** field, type **Okta Workflow Endpoint**.

4.  In the Endpoint field, we will put in a dummy endpoint:
    <http://localhost.com/api>

5.  After we activate the Okta workflows you will have to come back here
    and update the endpoints for the REST Message and POST API endpoint.

6.  Click **Update** to save your changes.

7.  Click on the **Okta Workflow Endpoint** REST Message. 
8.  Click on HTTP Request.


9.  Click **New**.


10. In the **Name** field, type **POST**.

11. From the **HTTP method** dropdown menu, select **POST**.

12. Add Okta workflow **Process Endpoint** api invoke endpoint in the **Endpoint** field. This will be something like <https://..oktapreview.com/api/flo/../invoke?clientToken=...>

13. Click **Submit**.

14. Click back into your **POST** HTTP Method.

15. Hover over the **Insert a new row text** under Name and press
    **Enter** on your keyboard.

16. Type in **Content-Type** and click the green check mark.


17. Under Value, click Enter and type in **application/json**. Click on
    the green check mark.

18. Under HTTP Query Parameters, find the **Content** field. Paste the
    following:

	`{"incidentNumber":"${incidentNumber}", "incidentMetadata":"${incidentMetadata}","incidentState":"${incidentState}"}`

19. Click **Update**.

20. You will be sent back to the main REST Message screen. Click on the
    HTTP **POST** Method you just created.

21. Scroll down to find Variable Substitutions. Click **New**.

22. In the **Name** field **incidentMetadata**.

23. In the **Test value** field, type **123**. This is a dummy value so
    that we can test the service in the future, if needed.

24. Create two other Variable Substitutions named:

> **incidentNumber**
>
> **incidentState**
>

25. Click on **Update** button to save the POST configuration. 
26. Click on **Update** at top right corner to save REST Message


### Create a Business Rule

1.  In the filter navigator, type **Business Rules** and choose **System
    Definition \> Business Rules** (Do not choose Metrics \> Business
    Rules.).


2.  Click on **New** to create a business rule.

3.  In the **Name** field, type **Okta Incident State Change to
    Resolved**.

4.  In the **Table** dropdown menu, select **Incident \[incident\]**.

5.  Mark the box next to **Advanced**.

6.  Scroll down to the **When to run** tab.

7.  Mark the box next to **Update**.

8.  Click on the **\-- choose field \--** dropdown menu and select
    **Incident state**.

9.  Click on the **\-- None \--** dropdown menu and select **Resolved**.

10. Click on the **Actions** tab,

11. Click on **\-- choose field \--** and select **State**. Click on
    **\-- None \--** and select **Resolved**.

12. Click on **\-- choose field \--** and select **Incident state**.
    Click on **\-- None \--** and select **Resolved**.


13. Click on the **Advanced** tab.

14. In the **Script** field, select all existing code. Paste in the
    following code:


	`(function executeRule(current, previous /*null when async*/) {`

	`var sm = new sn_ws.RESTMessageV2("Okta Workflow endpoint","post");`

	`//Might throw exception if message does not exist or not visible due`
	`to scope.`

	`sm.setStringParameter("incidentMetadata",current.short_description);`

	`sm.setStringParameter("incidentNumber",current.number);`

	`sm.setStringParameter("incidentState",current.approval);`

	`var response = sm.execute(); `

	`//Might throw exception if http connection timed out or some issue with sending request itself because of encryption/decryption of password.`

	`})(current, previous);`


15. Click **Submit**.

## Testing this flow
- Activate the user whose Department profile attribute is set to "Corellia" in Okta. 
- Approve and Resolve the ServiceNow Incident in ServiceNow.
- Check group membership is assigned to the user in Okta. 

## Limitations & Known Issues

- Keep in mind the [Okta Workflows System Limits](https://help.okta.com/en/prod/Content/Topics/Workflows/workflows-system-limits.htm) to limit the entries in the Okta table and the api requests to the Okta Workflow Invoke URL endpoint. 
- The Okta table is not intended to be an audit log - instead use Okta syslog and ServiceNow logs.
- This approval flow uses ServiceNow Incidents rather than ServiceNow Self Service Requests that some organizations may prefer for approval requests.
- The template is intended to be adapted for real-world scenarios. Group Ids to be assigned may, for example, be looked up in Okta Tables rather than hardcoded etc.  
