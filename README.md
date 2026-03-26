// Add these properties at the class level
currentQuestionIndex = 0;

// Add this getter (simple boolean)
get currentQuestions() {
    if (!this.resultData?.questions) return [];
    
    return this.resultData.questions.map((q, index) => ({
        ...q,
        isCurrent: index === this.currentQuestionIndex   // this works
    }));
}



<template if:true={resultData.questions}>
  <template for:each={currentQuestions} for:item="q" for:index="index">

    <!-- Only the current question will render -->
    <template if:true={q.isCurrent}>

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

        <!-- Your checkbox template (keep exactly as before) -->
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
</template>


renderedCallback() {
    if (this.isverifiedQuestionsAvailable && this.modalIsOpen) {
        setTimeout(() => {
            const firstInput = this.template.querySelector('[data-type="user-input"]');
            if (firstInput) {
                firstInput.focus();
            }
        }, 150);

        this.isverifiedQuestionsAvailable = false;
    }
}


goToNextQuestion() {
    if (this.currentQuestionIndex < this.resultData.questions.length - 1) {
        this.currentQuestionIndex++;
        // Focus the new question automatically
        setTimeout(() => {
            const currentInput = this.template.querySelector('[data-type="user-input"]');
            if (currentInput) currentInput.focus();
        }, 100);
    }
}


