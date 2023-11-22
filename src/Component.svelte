<script>
  import { getContext } from "svelte"
  import SuperCell from "../bb-component-SuperTableCell/lib/SuperTableCell/SuperCell.svelte"

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group")
  const formApi = formContext?.formApi;

  export let field
  export let fieldType

  let formField
  let formStep
  let fieldState
  let fieldApi
  let value

  let cellState
  let wrapperAnchor

  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(field, "string", 0, false, null, formStep);
  $: value = fieldState?.value
  $: showLabel = labelPos === "above" || false
  $: disabled = fieldState?.disabled

  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex-direction" : labelPos == "above" ? "column" : "row",
      "min-height": labelPos == "above" ? "3.85rem" : "2rem",
      gap: labelPos == "above" ? "0.1rem" : "0.85rem"
    }

  }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div class="superField"
  bind:this={wrapperAnchor}
  use:styleable={$component.styles}
  on:focus={cellState.focus}
  tabindex="0"
  >
    <label for="superCell" class="spectrum-FieldLabel spectrum-FieldLabel--sizeM superFieldLabel">{field}</label>
      <SuperCell
        bind:cellState 
        cellOptions={{}}
        value= ""
        fieldSchema={$formField?.fieldSchema || {} }
        editable = { true }
      />
</div>

<style>
.superField {
  display: flex;
  align-items: stretch;
  justify-content: stretch;
  min-width: 0;
}
.superFieldLabel {
  flex: auto;
  display: flex;
  align-items: center;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-height: 1.75rem;
}
</style>