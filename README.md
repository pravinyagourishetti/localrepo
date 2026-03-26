<template if:true={resultData.questions}>
  <template for:each={resultData.questions} for:item="q" for:index="index">
    <div key={q.id} class="question-wrapper slds-m-bottom_medium">
      
      <!-- Visible label (properly associated) -->
      <label 
        class="slds-form-element_label bold-label" 
        id={q.id}
        for={q.code}>
        {q.text}<span style="color: red;">*</span>
      </label>

      <!-- lightning-input – simplified, no extra live regions -->
      <lightning-input 
        type={q.type}
        date-style="short"
        id={q.code}
        name={q.id}
        required
        autocomplete="off"
        label={q.text}
        variant="label-hidden"
        class={q.dob}
        maxlength={q.maxlength}
        onchange={handleValidations}
        data-type="user-input"
        aria-labelledby={q.id}
        aria-required="true">
      </lightning-input>

      <!-- Checkboxes stay the same (no aria-live) -->
      <template for:each={q.autoFillCheckboxList} for:item="eachCheckBox">
        <div key={eachCheckBox.dob} class={eachCheckBox.className}>
          <lightning-input 
            class={eachCheckBox.inputClassName} 
            type="checkbox"
            onclick={handleCheckBoxOnChange}>
          </lightning-input>
          <label class="slds-form-element_label bold-label">
            {eachCheckBox.label}
          </label>
        </div>
      </template>

    </div>
  </template>
</template>


// After modal opens and questions are rendered
renderedCallback() {
  if (this.modalIsOpen && this.resultData?.questions?.length) {
    // Focus the very first input
    const firstInput = this.template.querySelector('lightning-input');
    if (firstInput) {
      firstInput.focus();
    }
  }
}



handleValidations(event) {
    const input = event.target;
    const fieldId = input.id || '';
    const value = input.value ? input.value.trim() : '';

    // Reset error state first (important!)
    input.setCustomValidity('');
    // Do NOT manually set aria-invalid or aria-errormessage here
    // lightning-input will manage it based on setCustomValidity

    if (fieldId.includes('LAST_4_SSN')) {
        input.value = value.replace(/[^0-9]/g, '');   // cleaned regex

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length === 4) {
            // valid case - do nothing (empty custom validity = valid)
        } else {
            input.setCustomValidity(FIELD_SSN_DIGIT);
        }
        input.reportValidity();
        this.checkValidity = input.validity.valid === false;   // better way
    }

    else if (fieldId.includes('DATE_OF_BIRTH')) {
        // your date formatting logic here...
        this.dobvalidation(event);   // keep your existing helper if needed
    }

    else if (fieldId.includes('PHONE_ON_PROFILE')) {
        input.value = value.replace(/\D/g, '');   // only digits

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length === 10) {
            // valid
        } else {
            input.setCustomValidity(FIELD_PHONE_DIGIT);
        }
        input.reportValidity();
        this.checkValidity = !input.validity.valid;
    }

    else if (fieldId.includes('ACCOUNT_NUMBER')) {
        input.value = value.replace(/[^0-9]/g, '');

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length === 16) {   // adjust length as per your requirement
            // valid
        } else {
            input.setCustomValidity(FIELD_ACC_NUMBER);
        }
        input.reportValidity();
        this.checkValidity = !input.validity.valid;
    }

    else if (fieldId.includes('MAGIC_PASSWORD')) {
        const isValidAlphaNum = handleAlphaNumericValidation(value);
        this.isDisplayUnidentifiedPrsn = false;   // fixed typo in your original

        if (value === '') {
            this.setValidationMessage(event, FIELD_NOT_BLANK, true);   // use your helper
            this.bypassButton = { ...this.bypassButton, disabled: false };
        } else if (isValidAlphaNum) {
            this.setValidationMessage(event, '', false);
        } else {
            this.setValidationMessage(event, this.labelUtility.constants.VerbalPwdValidationError, false);
            this.bypassButton = { ...this.bypassButton, disabled: true };
        }
    }

    // Final step - enable/disable Verify button
    this.validateVerifyButton(this.checkValidity);
}


setValidationMessage(event, message, isInvalid) {
    const input = event.target;
    input.setCustomValidity(message || '');
    input.reportValidity();
    this.checkValidity = isInvalid;
}





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
