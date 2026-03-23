<template>
    <div class="slds-modal slds-fade-in-open" role="dialog" aria-modal="true">
        <div class="slds-modal__container">

            <!-- HEADER -->
            <header class="slds-modal__header">
                <h2 id="modal-heading" class="slds-text-heading_medium">
                    Identity Verification Required
                </h2>
            </header>

            <!-- BODY -->
            <div class="slds-modal__content slds-p-around_medium">

                <!-- DATATABLE -->
                <lightning-datatable
                    key-field="id"
                    data={data}
                    columns={columns}
                    onrowaction={openCustomerLanding}
                    aria-labelledby="modal-heading">
                </lightning-datatable>

                <!-- APPROVE API MESSAGE -->
                <template if:true={isApproveApi}>
                    <div class="slds-box slds-box_small slds-m-top_small">
                        <p aria-live="polite">
                            <lightning-formatted-text
                                value="Approve API is called">
                            </lightning-formatted-text>
                        </p>
                    </div>
                </template>

                <!-- QUESTIONS -->
                <template if:true={resultData.questions}>
                    <div role="group" class="slds-m-top_medium">

                        <template for:each={resultData.questions} for:item="q">
                            <div key={q.id} class="slds-m-bottom_small">

                                <lightning-input
                                    label={q.label}
                                    value={q.value}
                                    data-id={q.id}
                                    onchange={handleInputChange}>
                                </lightning-input>

                            </div>
                        </template>

                    </div>
                </template>

            </div>

            <!-- FOOTER -->
            <footer class="slds-modal__footer">

                <!-- NORMAL FLOW -->
                <template if:false={agentLandingButton}>

                    <!-- UNIDENTIFIED BUTTON -->
                    <template if:true={isDisplayUnidentifiedBtn}>
                        <lightning-button
                            label={label.Unidentified_Prospect_Button}
                            title={label.Unidentified_Prospect_Button}
                            onclick={openworkflowtiav}
                            icon-name="utility:user"
                            variant="neutral"
                            class="slds-m-right_small">
                        </lightning-button>
                    </template>

                    <!-- DROPDOWN -->
                    <lightning-combobox
                        label="Call Dropped Reason"
                        value={value}
                        variant="label-hidden"
                        placeholder="Close Customer Locate"
                        options={options}
                        onchange={handleCallDropdownChange}
                        class="slds-m-right_small">
                    </lightning-combobox>

                    <!-- BYPASS BUTTON -->
                    <template if:true={showBypassBtn}>
                        <lightning-button
                            variant="brand"
                            label="ByPass"
                            onclick={handleBypassButton}
                            disabled={bypassButtonDisabled}
                            class="slds-m-right_small">
                        </lightning-button>
                    </template>

                    <!-- VERIFY BUTTON -->
                    <lightning-button
                        variant="brand"
                        label="Verify"
                        onclick={handleSubmit}
                        disabled={checkValidity}>
                    </lightning-button>

                </template>

                <!-- AGENT LANDING BUTTON -->
                <template if:true={agentLandingButton}>
                    <lightning-button
                        variant="brand"
                        label={btnAgentLandingText}
                        onclick={handleReloadButtuon}>
                    </lightning-button>
                </template>

            </footer>
        </div>
    </div>

    <!-- BACKDROP -->
    <div class="slds-backdrop slds-backdrop_open"></div>

    <!-- WORKFLOW POPUP -->
    <template if:true={showworkFlow}>
        <div class="slds-modal slds-fade-in-open">

            <div class="slds-modal__container">

                <lightning-icon
                    icon-name="utility:close"
                    size="small"
                    onclick={closeworkflowNav}
                    class="slds-m-around_small">
                </lightning-icon>

                <c-care_unidentifyworkflow
                    account-info={accountInfo}
                    unidentifyinfo={unidentifyinfo}
                    onclosemodalworkflow={closemodalworkflow}
                    ucontactid={ucontactid}
                    idnvobj={idnvobj}>
                </c-care_unidentifyworkflow>

            </div>
        </div>

        <div class="slds-backdrop slds-backdrop_open"></div>
    </template>

    <!-- STEP UP COMPONENT -->
    <template if:true={isStepUpApi}>
        <c-care_step-up-container
            onstepupvalidatesuccess={handleActivityResponse}
            ondecision={handleDecision}>
        </c-care_step-up-container>
    </template>

</template>


Below are the three final approaches you can include in your design document for this integration scenario (~5,000 records/day with Agent–Manager hierarchy validation).


---

Approach 1: Using Salesforce Standard REST APIs

Approach

In this approach, the integration logic is implemented on the AWS side, and Salesforce is used only for data access and record operations through standard REST APIs.

AWS receives the daily dataset containing Agent BRID, Manager BRID, and TSYS ID and processes each record through an integration service (such as AWS Lambda or another backend service). The AWS service communicates with Salesforce using Salesforce REST APIs.

The AWS service performs the required validation and hierarchy checks by making API calls to Salesforce.

The workflow operates as follows:

AWS queries Salesforce to find the Agent Contact using Agent BRID.

If the Agent Contact does not exist, AWS retrieves the Agent User record and creates a new Agent Contact using information from the User object.

AWS validates whether the Agent Contact is mapped to the correct Manager Contact.

AWS queries Salesforce to check whether the Manager Contact exists using Manager BRID.

If the Manager Contact exists, AWS updates the Agent Contact Reports To field.

If the Manager Contact does not exist, AWS queries the Manager User.

If the Manager User exists, AWS creates a Manager Contact using User information and sets its Reports To field to Default Contact Owner.

If neither Manager Contact nor Manager User exists, AWS updates the Agent Contact Reports To field with Default Contact Owner.

AWS also validates and updates the Agent User record, ensuring the ManagerId and TSYS ID fields are correct.


Pros

No custom Salesforce development required.

Uses Salesforce out-of-the-box REST APIs.

Business logic is centralized in the AWS integration layer.

Easier to modify integration logic without Salesforce deployments.


Limitations

Requires multiple API calls per record (queries, creates, updates).

For 5,000 records/day, this may generate 20,000–30,000 API calls daily.

Salesforce REST API returns maximum 2,000 records per query, so pagination must be implemented.

REST API request/response payload is limited to ~6 MB, requiring batching.

Increased network latency due to many HTTP requests.

Harder to maintain complex hierarchy logic outside Salesforce.



---

Approach 2: Using Custom Apex REST API (Recommended)

Approach

In this approach, a custom Apex REST service is created in Salesforce to process the integration logic internally.

AWS sends batches of records containing Agent BRID, Manager BRID, and TSYS ID to the Salesforce endpoint. Salesforce performs all validation and hierarchy processing using bulk Apex logic.

AWS sends the 5,000 daily records in smaller batches (e.g., 200–500 records) to avoid hitting Salesforce limits.

The Apex service performs the following:

Retrieves Agent Contacts using Agent BRID.

If an Agent Contact does not exist, retrieves the Agent User and creates a new Agent Contact using User data.

Validates whether the Agent Contact is mapped to the correct Manager Contact.

Checks whether the Manager Contact exists.

If the Manager Contact exists, updates the Agent Contact Reports To field.

If the Manager Contact does not exist, checks whether the Manager User exists.

If the Manager User exists, creates a Manager Contact using User data and assigns Default Contact Owner as Reports To.

If neither exists, assigns Default Contact Owner as the Agent Contact’s manager.

Updates the Agent User record, ensuring:

Correct ManagerId

Correct TSYS ID



All updates are performed using bulk SOQL queries and bulk DML operations.

Pros

Significantly fewer API calls compared to standard REST integrations.

Business logic handled within Salesforce, simplifying AWS implementation.

Supports complex validation and hierarchy logic easily.

Efficient bulk processing using Apex.

Reduced network overhead and latency.

Better transaction consistency.


Limitations

Requires custom Apex development.

Must operate within Salesforce governor limits:

100 SOQL queries

150 DML statements

10-second CPU time

6 MB heap size


Request payload must stay within ~6 MB REST limit.

Requires proper error handling and retry logic.



---

Approach 3: Using Salesforce Bulk API with Staging Object

Approach

In this approach, AWS uses Salesforce Bulk API v2 to upload the daily dataset containing Agent BRID, Manager BRID, and TSYS ID.

The records are first inserted into a custom staging object in Salesforce. After insertion, Apex processing logic reads the staging records and performs the required validation and updates.

The workflow operates as follows:

AWS prepares the 5,000 records in CSV format.

AWS creates a Bulk API job.

AWS uploads the data to Salesforce.

Salesforce inserts the records into a staging object (e.g., Agent_Manager_Staging__c).

An Apex trigger or asynchronous Apex process reads the staging records.

The Apex logic performs:

Agent Contact lookup

Manager Contact lookup

Manager User lookup

Contact creation if required

Updates to Reports To hierarchy

Updates to User ManagerId and TSYS ID


After processing completes, the staging records are deleted to free up storage.


Pros

Bulk API supports very large payloads (up to ~150 MB).

Requires very few API calls from AWS.

Efficient for large data ingestion jobs.

Staging object allows auditability and tracking of uploaded records.

Suitable for large-scale integrations.


Limitations

Requires additional staging object and Apex processing logic.

Integration becomes multi-step (data ingestion + processing).

Bulk API jobs are asynchronous, requiring job monitoring.

More complex architecture compared to REST-based integrations.

Overhead may be unnecessary for moderate volumes like 5,000 records/day.



---

Final Comparison

Parameter	Standard REST API	Apex REST API	Bulk API

Business Logic Location	AWS	Salesforce	Salesforce
API Calls	High	Low	Very Low
Data Volume Handling	Moderate	Good	Excellent
Complexity	Medium	Medium	High
Real-Time Processing	Yes	Yes	No (Async)
Best Use Case	Simple integrations	Complex integrations	Very large datasets



---



Problem Statement (Short)

An integration is required to synchronize Agent–Manager hierarchy data between AWS and Salesforce using the fields Agent BRID, Manager BRID, and TSYS ID received from AWS.

The Contact object represents agents and managers and maintains the reporting hierarchy through the Reports To field, while the User object maintains the system hierarchy through the Manager field.

The system must ensure that the Agent Contact is mapped to the correct Manager Contact. If the Agent Contact does not exist, it should be created using information from the corresponding User record. If the Manager Contact does not exist, the system must check for the Manager User and create a Manager Contact if necessary. If neither exists, the Default Contact Owner should be assigned as the manager.

Additionally, the Agent User record must reflect the correct Manager relationship and TSYS ID based on the data received from AWS.

The goal of this integration is to maintain accurate reporting hierarchy and user information in Salesforce aligned with AWS data.
