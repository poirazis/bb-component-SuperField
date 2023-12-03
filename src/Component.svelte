<script>
  import { getContext } from "svelte";
  import SuperCell from "../bb-component-SuperTableCell/lib/SuperTableCell/SuperCell.svelte";
  import {flip} from "svelte/animate";
  import {dndzone} from "svelte-dnd-action";

  const { styleable, builderStore, componentStore } = getContext("sdk");
  const component = getContext("component");

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");
  const labelPos = getContext("field-group");
  const labelWidth = getContext("field-group-label-width");
  const formApi = formContext?.formApi;

  export let stringField;
  export let numberField;
  export let booleanField;
  export let datetimeField;
  export let optionsField;
  export let linkField;
  export let arrayField;
  export let stringValidation;
  export let numberValidation;
  export let booleanValidation;
  export let datetimeValidation;
  export let optionsValidation;
  export let linkValidation;
  export let arrayValidation;

  export let frontButtons = [];
  export let frontButtonsQuiet;
  export let frontButtonsGrouping;
  export let frontButtonsSelected = [0];
  export let buttons = [];
  export let buttonsQuiet;
  export let buttonsGrouping;
  export let buttonsSelected = [0];

  export let fieldLabel;
  export let fieldType = "string";
  export let span = 6;
  export let inForm = false;
  export let repeatable = false

  let formField;
  let formStep;
  let fieldState;
  let fieldApi;
  let value;

  let cellStates = []
  let wrapperAnchor;
  let items = [ { id: 0, "name" : "0", value: "" } ]
  let flipDurationMs = 130

  function handleDndConsider(e) {
      items = e.detail.items;
  }

  function handleDndFinalize(e) {
    items = e.detail.items;
  }

  $: fieldName =
    fieldType == "string"
      ? stringField
      : fieldType == "number"
        ? numberField
        : fieldType == "boolean"
          ? booleanField
          : fieldType == "datetime"
            ? datetimeField
            : fieldType == "options"
              ? optionsField
              : fieldType == "link"
                ? linkField
                : fieldType == "array"
                  ? arrayField
                  : stringField;

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
      $builderStore.inBuilder &&
      $componentStore.selectedComponentPath?.includes($component.id) &&
      formContext &&
      !inForm
    ) {
      builderStore.actions.updateProp("inForm", true);
    } else if (
      $builderStore.inBuilder &&
      $componentStore.selectedComponentPath?.includes($component.id) &&
      inForm &&
      !formContext
    ) {
      builderStore.actions.updateProp("inForm", false);
    }
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      "flex-direction": labelPos == "left" ? "row" : "column",
      "max-height": labelPos == "left" ? "2rem" : ((items?.length * 2.25) + 2 )+ "rem",
      "min-height": labelPos == "left" ? "2rem" : ((items?.length * 2.25) + 2 )+ "rem",
      gap: labelPos == "left" ? "0.85rem" : "0rem",
      "grid-column": "span " + span,
      "--label-width":
        labelPos == "left" ? (labelWidth ? labelWidth : "6rem") : "auto",
    },
  };
 
  const buttonClick = (group, idx) => {
    if (group == "front" && frontButtonsGrouping == "single") {
      frontButtonsSelected = [idx];
    } else if (group == "end" && buttonsGrouping == "single") {
      buttonsSelected = [idx];
    }

    if (group == "front" && frontButtonsGrouping == "multi") {
      let i = frontButtonsSelected.indexOf(idx);
      if (i > -1) frontButtonsSelected.splice(i, 1);
      else frontButtonsSelected.push(idx);

      frontButtonsSelected = frontButtonsSelected;
    } else if (group == "end" && buttonsGrouping == "multi") {
      buttonsSelected = [idx];
    }

    if ( group == "repeater_add" ) {
      items = [...items, { id: idx + 1 , name: idx + 1}]
    } else if ( group == "repeater_remove" ) {
      if ( items.length > 1) {
      items.pop();
      items = items;
      }
    }
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
<div
  class="superField"
  bind:this={wrapperAnchor}
  use:styleable={$component.styles}  
>
  <label for="superCell" class="superFieldLabel" class:bound={formContext}>
    {fieldLabel || fieldName || "Unamed Field"}
  </label>
  
  <div class="cell-items" use:dndzone="{{items, flipDurationMs, dropTargetStyle: {}}}" on:consider="{handleDndConsider}" on:finalize="{handleDndFinalize}" >

    {#each items as item , item_idx (item.id)}
      <div class="inline-cell">
        {#if repeatable}
          <div class="dragHandle">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-grip-vertical"><circle cx="9" cy="12" r="1"/><circle cx="9" cy="5" r="1"/><circle cx="9" cy="19" r="1"/><circle cx="15" cy="12" r="1"/><circle cx="15" cy="5" r="1"/><circle cx="15" cy="19" r="1"/></svg>
          </div>
        {/if}

        {#if frontButtons?.length}
          <div
            class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
            class:spectrum-ActionGroup--quiet={frontButtonsQuiet}
          >
            {#each frontButtons as button, idx}
              <button
                on:click={(e) => buttonClick("front", idx)}
                class="spectrum-ActionButton spectrum-ActionButton--sizeM"
                class:spectrum-ActionButton--quiet={frontButtonsQuiet ||
                  button.quiet}
                class:warning={button.type == "warning"}
                class:cta={button.type == "cta"}
                class:disabled={button.disabled}
                class:is-selected={frontButtonsGrouping &&
                  frontButtonsSelected.includes(idx)}
              >
                <span class="spectrum-ActionButton-label">{@html button?.text}</span
                >
              </button>
            {/each}
          </div>
        {/if}

        <div class="inline-cells" on:focus={cellStates[item.id].focus} tabindex="0">
          <SuperCell
            bind:cellState={cellStates[item.id]}
            cellOptions={{}}
            value={items[item_idx].value}
            fieldSchema={{ type: fieldType }}
            editable={true}
            id={"sp_cell_" + item_idx}
            on:change={(e) => items[item_idx].value = e.detail}
          />
        </div>

        {#if buttons?.length}
          <div
            class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
            class:spectrum-ActionGroup--quiet={buttonsQuiet}
          >
            {#each buttons as button, idx}
              <button
                class="spectrum-ActionButton spectrum-ActionButton--sizeM"
                on:click={(e) => buttonClick("end", idx)}
                class:spectrum-ActionButton--quiet={buttonsQuiet || button.quiet}
                class:warning={button.type == "warning"}
                class:cta={button.type == "cta"}
                class:disabled={button.disabled}
                class:is-selected={buttonsGrouping && buttonsSelected.includes(idx)}
              >
                <span class="spectrum-ActionButton-label">{@html button?.text}</span
                >
              </button>
            {/each}
          </div>
        {/if}

        {#if repeatable}
          <div
            class="spectrum-ActionGroup spectrum-ActionGroup--compact spectrum-ActionGroup--sizeM"
            class:spectrum-ActionGroup--quiet={buttonsQuiet}
          >

            {#if items.length > 1}
              <button
                class="spectrum-ActionButton spectrum-ActionButton--sizeM"
                on:click={(e) => buttonClick("repeater_remove", item.id)}
                class:spectrum-ActionButton--quiet={buttonsQuiet}
                class:warning={1}
              >
                <span class="spectrum-ActionButton-label">
                  <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-x"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
                </span>
              </button>
            {/if}
            {#if items.length == 1 || item_idx == (items.length - 1) }
            <button
              class="spectrum-ActionButton spectrum-ActionButton--sizeM"
              on:click={(e) => buttonClick("repeater_add", item.id)}
              class:spectrum-ActionButton--quiet={buttonsQuiet}
            >
              <span class="spectrum-ActionButton-label">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-plus"><path d="M5 12h14"/><path d="M12 5v14"/></svg>
              </span>
            </button>
          {/if}
          </div>
        {/if}

        </div>
    {/each}
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

  .cell-items {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .inline-cell {
    flex: 1;
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    gap: 0.25rem;
  }

  .inline-cells {
    flex: 1;
    display: flex;
    justify-items: stretch;
    height: 2rem;
  }
  .warning {
    color: var(--spectrum-global-color-red-500);
    border-color: var(--spectrum-global-color-red-500);
  }

  .spectrum-ActionButton--quiet.warning {
    color: var(--spectrum-global-color-red-500);
    border: none;
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

  :global(.spectrum-ActionGroup > button > span > svg) {
    width: 14px;
    height: 14px;
  }

  .superFieldLabel.bound {
    gap: 0.5rem;
  }

  .dragHandle {
    display: flex;
    align-items: center;
    height: 100%;
    color: var(--spectrum-global-color-gray-600);
  }
  .dragHandle:hover {
    color: var(--spectrum-global-color-gray-900);
  }
</style>
