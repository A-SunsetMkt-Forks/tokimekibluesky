<script lang="ts">
    import {_} from "svelte-i18n";
    import { page } from '$app/stores';
    import ListView from "./ListView.svelte";
    import PageModal from "$lib/components/ui/PageModal.svelte";
    import type { Snapshot } from './$types';
    import {settings} from "$lib/stores";
    import {tick} from "svelte";

    let title = $state('');

    export const snapshot: Snapshot = {
        capture: () => [$settings.design.layout === 'decks' ? document.querySelector('.modal-page-content').scrollTop : document.querySelector(':root').scrollTop],
        restore: (value) => {
          [scrollY] = value;

          tick().then(() => {
            if ($settings.design.layout === 'decks') {
              document.querySelector('.modal-page-content').scroll(0, scrollY);
            } else {
              document.querySelector(':root').scroll(0, scrollY);
            }
          })
        }
    };
</script>

<PageModal>
  <div class="column-heading">
    <div class="column-heading__buttons">
      <button class="settings-back" onclick={() => {history.back()}}>
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="var(--text-color-1)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-arrow-left"><path d="m12 19-7-7 7-7"/><path d="M19 12H5"/></svg>
      </button>
    </div>

    <h1 class="column-heading__title">{$_('lists')} - {title}</h1>

    <div class="column-heading__buttons column-heading__buttons--right">
      <a class="settings-back" href="/">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="var(--text-color-1)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-x"><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>
      </a>
    </div>
  </div>

  {#key $page.params.id}
    <ListView id={$page.params.id} handle={$page.params.handle} bind:title={title}></ListView>
  {/key}
</PageModal>