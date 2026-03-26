<template if:true={resultData.questions}>

  <!-- Keep your original Call Intent block -->
  <div dir="rtl">
    <!-- USCBCOLLCT-34194 Start-->
    <template if:true={intentvisibility}>
      <div class="containterInt">
        <label>Call Intent: </label>
      </div>
      <label>{callintentuIvalue}</label>
      <br />
    </template>
    <!-- USCBCOLLCT-34194 End-->
  </div>

  <!-- Clean questions - no extra aria-live or duplicate attributes -->
  <template for:each={resultData.questions} for:item="q" for:index="index">
    <div key={q.id}>

      <div style="display: flex;">
        <label 
          class="slds-form-element_label bold-label" 
          id={q.id}
          for={q.code}>
          {q.text}<span style="color: red;">*</span>
        </label>
      </div>

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
        aria-required="true">
      </lightning-input>

      <!-- Your original checkboxes unchanged -->
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

renderedCallback() {
    if (this.isVerifiedQuestionsAvailable && this.modalIsOpen) {   // adjusted flag name to match your code
        setTimeout(() => {
            const userInputs = this.template.querySelectorAll('[data-type="user-input"]');
            if (userInputs && userInputs.length > 0) {
                userInputs[0].focus();   // focus first input
            }
        }, 200);   // delay helps in modals

        this.isVerifiedQuestionsAvailable = false;
    }
}

handleValidations(event) {
    const input = event.target;
    const fieldId = input.id || '';
    let value = (input.value || '').trim();

    // Reset error first
    input.setCustomValidity('');
    input.reportValidity();

    if (fieldId.includes('LAST_4_SSN')) {
        value = value.replace(/[^0-9]/g, '');
        input.value = value;

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length !== 4) {
            input.setCustomValidity(FIELD_SSN_DIGIT);
        }
        // valid case = empty custom validity
        input.reportValidity();
        this.checkValidity = !input.validity.valid;
    } 
    else if (fieldId.includes('DATE_OF_BIRTH')) {
        // your existing date logic here...
        this.dobvalidation(event);
        // At the end of dobvalidation, use setCustomValidity + reportValidity
    } 
    else if (fieldId.includes('PHONE_ON_PROFILE')) {
        value = value.replace(/\D/g, '');
        input.value = value;

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length !== 10) {
            input.setCustomValidity(FIELD_PHONE_DIGIT);
        }
        input.reportValidity();
        this.checkValidity = !input.validity.valid;
    } 
    else if (fieldId.includes('ACCOUNT_NUMBER')) {
        value = value.replace(/[^0-9]/g, '');
        input.value = value;

        if (value === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (value.length !== 16) {
            input.setCustomValidity(FIELD_ACC_NUMBER);
        }
        input.reportValidity();
        this.checkValidity = !input.validity.valid;
    } 
    else if (fieldId.includes('MAGIC_PASSWORD')) {
        let magicPasswordValue = value;
        let magicPasswordCheck = handleAlphaNumericValidation(magicPasswordValue);
        this.isDisplayUnidentifiedPrsn = false;   // fixed typo

        if (magicPasswordValue === '') {
            input.setCustomValidity(FIELD_NOT_BLANK);
        } else if (!magicPasswordCheck) {
            input.setCustomValidity(this.labelUtility?.constants?.VerbalPwdValidationError || 'Invalid password');
        }
        input.reportValidity();
        this.checkValidity = !input.validity.valid;

        this.bypassButton = { ...this.bypassButton, disabled: magicPasswordValue === '' };
    }

    this.validateVerifyButton(this.checkValidity);
}

<div class="sids-modal content spinnerPosition" id="modal-content-id-1">   <!-- remove aria-live="polite" -->

