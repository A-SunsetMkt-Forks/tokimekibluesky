<script lang="ts">
  import {_} from "svelte-i18n";
  import {agent} from "$lib/stores";
  import OfficialListItem from "$lib/components/list/OfficialListItem.svelte";
  import {isDid} from "$lib/util";
  import {defaultDeckSettings} from "$lib/components/deck/defaultDeckSettings";
  import DeckRow from "../../../../DeckRow.svelte";
  import {getColumnState} from "$lib/classes/columnState.svelte";

  interface Props {
    id: any;
    handle: any;
    title?: string;
    isModerationList?: boolean;
    isMute?: boolean;
    isBlock?: any;
  }

  let {
    id,
    handle,
    title = $bindable(''),
    isModerationList = $bindable(false),
    isMute = $bindable(false),
    isBlock = $bindable(undefined)
  }: Props = $props();

  const columnState = getColumnState(true);

  let timeline = [];
  let cursor = undefined;
  let feed;
  let did = $state('');
  let isButtonsDisable = $state(false);

  if (isDid(handle)) {
      did = handle;

      if (!columnState.hasColumn('list_' + id)) {
          columnState.add({
              id: 'list_' + id,
              algorithm: {
                  algorithm: 'at://' + did + '/app.bsky.graph.list/' + id,
                  type: 'officialList',
                  name: '',
              },
              style: 'default',
              settings: defaultDeckSettings,
              did: $agent.did(),
              handle: $agent.handle(),
              data: {
                  feed: [],
                  cursor: '',
              }
          });
      }
  } else {
      $agent.agent.api.com.atproto.identity.resolveHandle({handle: handle})
          .then(value => {
              did = value.data.did;

              if (!columnState.hasColumn('list_' + id)) {
                  columnState.add({
                      id: 'list_' + id,
                      algorithm: {
                          algorithm: 'at://' + did + '/app.bsky.graph.list/' + id,
                          type: 'officialList',
                          name: '',
                      },
                      style: 'default',
                      settings: defaultDeckSettings,
                      did: $agent.did(),
                      handle: $agent.handle(),
                      data: {
                          feed: [],
                          cursor: '',
                      }
                  });
              }
          })
          .catch(e => {
              console.log(e);
          });
  }

  async function muteList() {
      isButtonsDisable = true;

      try {
          const res = await $agent.agent.api.app.bsky.graph.muteActorList({list: 'at://' + did + '/app.bsky.graph.list/' + id});
          isMute = true;
      } catch (e) {
          console.error(e);
      }

      isButtonsDisable = false;
  }

  async function unMuteList() {
      isButtonsDisable = true;

      try {
          const res = await $agent.agent.api.app.bsky.graph.unmuteActorList({list: 'at://' + did + '/app.bsky.graph.list/' + id});
          isMute = false;
      } catch (e) {
          console.error(e);
      }

      isButtonsDisable = false;
  }

  async function blockList() {
      isButtonsDisable = true;

      try {
          const res = await $agent.agent.api.app.bsky.graph.listblock.create(
              {
                  repo: $agent.did()
              },
              {
                  subject: 'at://' + did + '/app.bsky.graph.list/' + id,
                  createdAt: new Date().toISOString(),
              },
          );
          isBlock = res.uri;
      } catch (e) {
          console.error(e);
      }

      isButtonsDisable = false;
  }

  async function unBlockList() {
      isButtonsDisable = true;

      try {
          const res = await $agent.agent.api.app.bsky.graph.listblock.delete(
              {
                  repo: $agent.did(),
                  rkey: isBlock.split('/').slice(-1)[0],
              },
          );
          isBlock = undefined;
      } catch (e) {
          console.error(e);
      }

      isButtonsDisable = false;
  }
</script>

{#if did}
  <div class="list-view-list-item">
    <OfficialListItem uri={'at://' + did + '/app.bsky.graph.list/' + id} bind:title={title} bind:isModerationList={isModerationList} bind:isMute={isMute} bind:isBlock={isBlock}></OfficialListItem>
  </div>

  {#if !isModerationList}
    {#if (columnState.hasColumn('list_' + id))}
      <DeckRow index={columnState.getColumnIndex('list_' + id)} isJunk={true} name={title}></DeckRow>
    {/if}
  {:else}
    <div class="mod-list-cover">
      <h2 class="mod-list-cover__title">{$_('mod_list_cover_title')}</h2>
      <p class="mod-list-cover__text">{$_('mod_list_cover_text')}</p>

      <div class="mod-list-buttons">
        {#if isMute}
          <button class="button button--sm button--border button--with-icon" onclick={unMuteList} disabled={isButtonsDisable}><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle-off"><path d="M20.5 14.9A9 9 0 0 0 9.1 3.5"/><path d="m2 2 20 20"/><path d="M5.6 5.6C3 8.3 2.2 12.5 4 16l-2 6 6-2c3.4 1.8 7.6 1.1 10.3-1.7"/></svg>{$_('button_unmute')}
          </button>
        {:else}
          <button class="button button--sm button--with-icon" onclick={muteList} disabled={isButtonsDisable}><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-message-circle-off"><path d="M20.5 14.9A9 9 0 0 0 9.1 3.5"/><path d="m2 2 20 20"/><path d="M5.6 5.6C3 8.3 2.2 12.5 4 16l-2 6 6-2c3.4 1.8 7.6 1.1 10.3-1.7"/></svg>{$_('button_mute')}
          </button>
        {/if}

        {#if isBlock}
          <button class="button button--sm button--border button--with-icon" onclick={unBlockList} disabled={isButtonsDisable}><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-shield-ban"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10"/><path d="m4 5 14 12"/></svg>{$_('button_unblock')}
          </button>
        {:else}
          <button class="button button--sm button--with-icon" onclick={blockList} disabled={isButtonsDisable}><svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-shield-ban"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10"/><path d="m4 5 14 12"/></svg>{$_('button_block')}
          </button>
        {/if}
      </div>
    </div>
  {/if}
{/if}

<style lang="postcss">
    .list-view-list-item {
        padding: 8px 0;
        border-bottom: 1px solid var(--border-color-2);
    }

    .mod-list-cover {
        padding: 16px;
        text-align: center;
        color: var(--text-color-3);

        &__title {
          font-size: 20px;
          margin-bottom: 8px;

          @media (max-width: 767px) {
              font-size: 18px;
          }
      }

      &__text {
          font-size: 14px;
      }
  }

  .mod-list-buttons {
      margin-top: 16px;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 8px 16px;
  }
</style>