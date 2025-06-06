<script lang="ts">
    import type { ClientRectObject, VirtualElement } from 'svelte-floating-ui/core'
    import { offset, flip } from "svelte-floating-ui/dom";
    import { createFloatingActions } from 'svelte-floating-ui'

    let { props } = $props();
    let selectedIndex = $state(0);
    const _props = $state(props);

    const [floatingRef, floatingContent] = createFloatingActions({
        strategy: 'fixed',
        placement: 'bottom-start',
        middleware: [
            offset(8),
            flip(),
        ]
    });

    const virtualElement: VirtualElement = {getBoundingClientRect(): ClientRectObject {
            return {
                x: 0,
                y: 0,
                top: _props.clientRect()?.top,
                left: _props.clientRect()?.left,
                bottom: 0,
                right: 0,
                width: _props.clientRect()?.width,
                height: _props.clientRect()?.height,
            }
        }}
    floatingRef(virtualElement);

    function selectItem(index) {
        if (!props?.items) {
            return false;
        }

        const item = props.items[index];

        if (item) {
            props.command({id: item});
        }
    }

    export function handleKeyDown({ event }) {
        if (event.key === 'ArrowUp') {
            if (_props?.items) {
                handleArrowUp();
                return true;
            }
        }

        if (event.key === 'ArrowDown') {
            if (_props?.items) {
                handleArrowDown();
                return true;
            }
        }

        if (event.key === 'Enter' && !event.ctrlKey) {
            handleEnter();
            if (_props?.items) {
                return true;
            }
        }
    }

    function handleArrowUp() {
        selectedIndex = ((selectedIndex + props.items.length) - 1) % props.items.length
    }

    function handleArrowDown() {
        selectedIndex = (selectedIndex + 1) % props.items.length
    }

    function handleEnter() {
        selectItem(selectedIndex);
    }
</script>

{#if props?.items}
  <div class="hashtag-list" use:floatingContent>
      {#each props.items as item, index}
        <button class="hashtag-list__item" class:hashtag-list__item--selected={index === selectedIndex} onclick={() => {selectItem(index)}}>
          <span class="hashtag-list__name">#{item}</span>
        </button>
      {/each}
  </div>
{/if}

<style lang="postcss">
    .hashtag-list {
        display: flex;
        flex-direction: column;
        background-color: var(--bg-color-1);
        padding: 8px 16px;
        z-index: 100;
        border-radius: var(--border-radius-3);
        box-shadow: 0 0 10px var(--box-shadow-color-2);

        &__item {
            color: var(--text-color-1);
            letter-spacing: .025em;
            white-space: nowrap;
            padding: 8px;
            width: 100%;
            text-align: left;
            border-radius: var(--border-radius-2);

            &--selected {
                background-color: var(--bg-color-2);
            }

            &:hover {
                background-color: var(--bg-color-2);
            }
        }

        &__name {
            font-weight: bold;
        }
    }
</style>