<script>
  import { getContext } from "svelte";
  import SuperCell from "../bb-component-SuperTableCell/lib/SuperTableCell/SuperCell.svelte";

  const { styleable, builderStore, componentStore } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const formApi = formContext?.formApi;

  export let stringField;
  export let numberField
  export let booleanField
  export let datetimeField
  export let optionsField
  export let linkField
  export let arrayField
  export let stringValidation;
  export let numberValidation
  export let booleanValidation
  export let datetimeValidation
  export let optionsValidation
  export let linkValidation
  export let arrayValidation

  export let frontButtons = []
  export let frontButtonsQuiet
  export let buttons = []
  export let buttonsQuiet

  export let fieldLabel
  export let fieldType = "string";
  export let span = 6;
  export let inForm = false

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let value;

  let cellState;
  let wrapperAnchor;

  $: fieldName =  fieldType == "string" ? stringField
                : fieldType == "number" ? numberField
                : fieldType == "boolean" ? booleanField
                : fieldType == "datetime" ? datetimeField
                : fieldType == "options" ? optionsField
                : fieldType == "link" ? linkField
                : fieldType == "array" ? arrayField 
                : stringField

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    fieldName,
    fieldType,
    undefined,
    false,
    null,
    formStep
  );
  $: value = fieldState?.value;
  $: showLabel = labelPos === "above" || false;
  $: disabled = fieldState?.disabled;

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: {
    if (
      $builderStore.inBuilder
      && $componentStore.selectedComponentPath?.includes($component.id)
      && formContext && !inForm
    ) {
      builderStore.actions.updateProp("inForm", true)
    } else if (
      $builderStore.inBuilder
      && $componentStore.selectedComponentPath?.includes($component.id)
      && inForm && !formContext
    ) {
      builderStore.actions.updateProp("inForm", false)
    }
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex-direction": labelPos == "left" ? "row" : "column",
      "max-height": labelPos == "left" ? "2rem" : "4rem",
      "min-height": labelPos == "left" ? "2rem" : "4rem",
      gap: labelPos == "left" ? "0.85rem" : "0rem",
      "grid-column": "span " + span,
      "--label-width": labelPos == "left" ? labelWidth ? labelWidth : "6rem" : "auto"
    },
  };

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div
  class="superField"
  bind:this={wrapperAnchor}
  use:styleable={$component.styles}
  on:focus={cellState.focus}
  tabindex="0"
>
    <label
      for="superCell"
      class="superFieldLabel"
      class:bound={formContext}
    >
      {fieldLabel || fieldName || "Unamed Field"}
    </label>

    <div class="inline-buttons">
      <div 
        class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        class:spectrum-ActionGroup--quiet={frontButtonsQuiet}
      >
        {#each frontButtons as button, idx }
          <button class="spectrum-ActionButton spectrum-ActionButton--sizeM"
          class:spectrum-ActionButton--quiet={frontButtonsQuiet || button.quiet }
          class:warning={button.type == "warning"}
          class:cta={button.type == "cta"}
          class:disabled={button.disabled}
          >
            <span class="spectrum-ActionButton-label">{@html button?.text}</span>
          </button>
        {/each}
      </div>

      <SuperCell
        bind:cellState
        cellOptions={{}}
        value=""
        fieldSchema={{ type: fieldType }}
        editable={true}
        id="superCell"
      />

      <div 
        class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
        class:spectrum-ActionGroup--quiet={buttonsQuiet}
      >
        {#each buttons as button, idx }
          <button class="spectrum-ActionButton spectrum-ActionButton--sizeM"
          class:spectrum-ActionButton--quiet={buttonsQuiet || button.quiet }
          class:warning={button.type == "warning"}
          class:cta={button.type == "cta"}
          class:disabled={button.disabled}
          >
            <span class="spectrum-ActionButton-label">{@html button?.text}</span>
          </button>
        {/each}
      </div>
    </div>
</div>

<style>
  .superField {
    display: flex;
    align-items: stretch;
    justify-content: stretch;
    min-width: 0;
  }
  .superFieldLabel {
    display: flex;
    align-items: center;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    min-width: var(--label-width);
    max-width: var(--label-width);
    font-size: 14px;
    line-height: 2rem;
    font-weight: 400;
    color: var(--spectrum-global-color-gray-700);
  }

  .warning {
    color: var(--spectrum-global-color-red-500);
    border-color: var(--spectrum-global-color-red-500);
  }
  .warning:hover {
    color: #000;
    background-color: var(--spectrum-global-color-red-500);
  }
  .cta {
    color: #fff;
    background-color: var(--primaryColor);
  }

  .disabled {
    color: var(--spectrum-global-color-gray-600); 
    background-color: var(--spectrum-global-color-gray-200);
  }

  .inline-buttons {
    height: 2rem;
    display: flex;
    justify-content: stretch;
    align-items: stretch;
  }

 :global(.inline-buttons > .spectrum-ActionGroup > button > span > svg ) {
    width: 14px;
    height: 14px;
  }

  .superFieldLabel.bound {
    gap: 0.5rem;
  }
</style>
