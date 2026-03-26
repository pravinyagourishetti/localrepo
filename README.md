<template if:true={resultData.questions}>

  <!-- Keep your original intent block exactly as it was -->
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

  <!-- Questions - restored to your original structure (no new wrappers) -->
  <template for:each={resultData.questions} for:item="q" for:index="index">
    <div key={q.id}>   <!-- removed aria-live="polite" -->

      <div style="display: flex;">
        <label 
          class="slds-form-element_label bold-label" 
          id={q.id}
          for={q.code}>   <!-- added for= to properly link label to input -->
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
        <!-- Removed: aria-label, aria-live, aria-valuemax, aria-labelledby, aria-describedby -->
      </lightning-input>

      <!-- Your checkboxes - exactly as before -->
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
    if (this.isverifiedQuestionsAvailable && this.modalIsOpen) {
        setTimeout(() => {
            const firstInput = this.template.querySelector('[data-type="user-input"]');
            if (firstInput) {
                firstInput.focus();
            }
        }, 200);

        this.isverifiedQuestionsAvailable = false;
    }
}

