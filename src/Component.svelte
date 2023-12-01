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

  export let thelabel
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

  $: fieldName = fieldType == "string" ? stringField
                : fieldType == "number" ? numberField
                : fieldType == "boolean" ? booleanField
                : fieldType == "datetiem" ? datetimeField
                : fieldType == "options" ? optionsField
                : fieldType == "link" ? linkField
                : fieldType == "array" ? arrayField
                : "Field"

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    fieldName,
    fieldType,
    0,
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
      "--label-width": labelWidth ? labelWidth : "5rem",
    },
  };

  $: console.log(labelPos)
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
    {thelabel || fieldName || "Unamed Field"}
    {#if formContext && $builderStore.inBuilder}
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="10"
        height="10"
        viewBox="0 0 24 24"
        fill="none"
        stroke="var(--spectrum-global-color-blue-500)"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="lucide lucide-database-zap"
        ><ellipse cx="12" cy="5" rx="9" ry="3" /><path
          d="M3 5V19A9 3 0 0 0 15 21.84"
        /><path d="M21 5V8" /><path d="M21 12L18 17H22L19 22" /><path
          d="M3 12A9 3 0 0 0 14.59 14.87"
        /></svg
      >
    {/if}
  </label>
  <SuperCell
    bind:cellState
    cellOptions={{}}
    value=""
    fieldSchema={{ type: fieldType }}
    editable={true}
  />
</div>

<style>
  .superField {
    display: flex;
    align-items: stretch;
    justify-content: stretch;
    min-width: 0;
    min-height: 4rem;
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
    font-weight: 400;
    color: var(--spectrum-global-color-gray-700);
  }

  .superFieldLabel.bound {
    gap: 0.5rem;
  }
</style>
