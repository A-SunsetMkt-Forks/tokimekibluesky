<script lang="ts">
  import {postState} from "$lib/classes/postState.svelte";
  import {publishState} from "$lib/classes/publishState.svelte";
  import {modalState} from "$lib/classes/modalState.svelte";
  interface Props {
    post: any;
    embeddingDisabled?: boolean;
  }

  let { post, embeddingDisabled = false }: Props = $props();

  function handleClick() {
      postState.quote = post;
      postState.quotePulse = Symbol();
      publishState.show = true;
      modalState.isVideoModalOpen = false;
      modalState.isMediaModalOpen = false;
  }
</script>

<button class="timeline-reaction__item timeline-reaction__item--quote" onclick={handleClick} aria-label="Quote" disabled={embeddingDisabled}>
  <span class="timeline-reaction__icon">
    <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="var(--timeline-reaction-like-icon-color)" stroke="none" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-quote"><path d="M3 21c3 0 7-1 7-8V5c0-1.25-.756-2.017-2-2H4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2 1 0 1 0 1 1v1c0 1-1 2-2 2s-1 .008-1 1.031V20c0 1 0 1 1 1z"/><path d="M15 21c3 0 7-1 7-8V5c0-1.25-.757-2.017-2-2h-4c-1.25 0-2 .75-2 1.972V11c0 1.25.75 2 2 2h.75c0 2.25.25 4-2.75 4v3c0 1 0 1 1 1z"/></svg>
  </span>
</button>

<style lang="postcss">
    .timeline-reaction__item {
        &:not(:disabled) {
            &:hover {
                @media (min-width: 768px) {
                    color: var(--timeline-reaction-like-icon-hover-color);

                    .timeline-reaction__icon::after {
                        background-color: var(--timeline-reaction-like-hover-bg-color);
                    }

                    svg {
                        fill: var(--timeline-reaction-like-icon-hover-color);
                    }
                }
            }
        }

        &:disabled {
            opacity: .4;
        }
    }
</style>