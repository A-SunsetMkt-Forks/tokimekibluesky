<script lang="ts">
  import { IsInViewport } from "runed";

  interface Props {
    video: any;
    isLocal?: boolean;
    isTok: boolean;
  }

  let { video, isLocal = false, isTok = false }: Props = $props();
  let el: HTMLElement | undefined = $state();
  const inView = new IsInViewport(() => el, {
    rootMargin: '500px 500px 500px 500px',
  });
</script>

<div class="timeline-video-wrap" class:timeline-video-wrap--tok={isTok} style="--video-width: {video?.aspectRatio?.width}; --video-height: {video?.aspectRatio?.height}" bind:this={el}>
  {#if inView.current}
    {#await import('$lib/components/video/VideoPlayer.svelte') then { default: VideoPlayer }}
      <VideoPlayer
        src={isLocal ? { src: video?.blob, type: 'video/object' } : video?.playlist}
        poster={video?.thumbnail}
        {isTok}
      ></VideoPlayer>
    {/await}
  {/if}
</div>

<style lang="postcss">
  .timeline-video-wrap {
      aspect-ratio: var(--video-width, 16) / var(--video-height, 9);
      max-height: 600px;
      margin-top: 16px;

      &--tok {
          margin-top: 0;
          aspect-ratio: auto;
          max-height: initial;
          height: 100%;
      }

      @container timeline-item (max-width: 345px) {
          margin-left: calc((var(--avatar-size) + var(--avatar-gap)) * -1);
      }
  }
</style>