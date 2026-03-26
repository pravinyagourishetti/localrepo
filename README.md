<template if:true={resultData.questions}>
  <template for:each={resultData.questions} for:item="q" for:index="index">
    <div key={q.id} class="question-wrapper slds-m-bottom_medium">

      <label class="slds-form-element_label bold-label" id={q.id}>
        {q.text}<span style="color: red;">*</span>
      </label>

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

      <!-- Your checkbox template - keep exactly as you had it -->
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
        }, 200);   // slightly longer delay often helps in modals

        this.isverifiedQuestionsAvailable = false;
    }
}
