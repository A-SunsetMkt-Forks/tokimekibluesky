<script lang="ts">
    import {_} from "svelte-i18n";
    import {Heart, Repeat2, Star, Pencil} from "lucide-svelte";
    import Avatar from "../../../routes/(app)/Avatar.svelte";
    import ProfileCardWrapper from "../../../routes/(app)/ProfileCardWrapper.svelte";
    import {getReasonText} from "$lib/components/notification/notificationUtil";
    import {AppBskyEmbedImages} from "@atproto/api";
    import Images from "../../../routes/(app)/Images.svelte";
    import LikesModal from "$lib/components/thread/LikesModal.svelte";
    import RepostsModal from "$lib/components/thread/RepostsModal.svelte";
    import {didHint, junkAgentDid, settings} from "$lib/stores";
    import FeedEmbed from "$lib/components/feeds/FeedEmbed.svelte";
    import {defaultDeckSettings} from "$lib/components/deck/defaultDeckSettings";
    import {goto} from "$app/navigation";
    import {getColumnState} from "$lib/classes/columnState.svelte";

    let { item, post, _agent } = $props();

    const junkColumnState = getColumnState(true);

    let isLikesOpen = $state(false);
    let isRepostsOpen = $state(false);

    function handleClick() {
        if (item.reason === 'like') {
            isLikesOpen = true;
        }

        if (item.reason === 'repost') {
            isRepostsOpen = true;
        }
    }

    function handlePostClick(e) {
        e.preventDefault();

        const rkey = post.uri.split('/').slice(-1)[0];
        const uri = '/profile/' + post.author.handle + '/post/' + rkey;

        if (uri === location.pathname) {
            return false;
        }

        if (!junkColumnState.hasColumn('thread_' + rkey)) {
            junkColumnState.add({
                id: 'thread_' + rkey,
                algorithm: {
                    algorithm: 'at://' + post.author.did + '/app.bsky.feed.post/' + rkey,
                    type: 'thread',
                    name: 'Thread',
                },
                style: 'default',
                settings: defaultDeckSettings,
                did: _agent.did(),
                handle: _agent.handle(),
                data: {
                    feed: [{
                        post: post,
                    }],
                    cursor: '',
                }
            });
        }

        junkAgentDid.set(_agent.did());
        didHint.set(post.author.did);
        goto(uri);
    }
</script>

<article class="notifications-item notifications-item--reaction notifications-item--{item.reason}" class:notifications-item--bubble={$settings?.design?.bubbleTimeline}>
    <div class="notification-column">
        <div class="notification-column__icons">
            {#if (item.reason === 'repost' || item.reason === 'repost-via-repost')}
                <button class="notification-icon notification-icon--repost" onclick={handleClick}>
                    <Repeat2 color="var(--timeline-reaction-reposted-icon-color)" size="18"></Repeat2>
                </button>
            {/if}

            {#if (item.reason === 'like' || item.reason === 'like-via-repost')}
                <button class="notification-icon notification-icon--like" onclick={handleClick}>
                    {#if ($settings?.design?.reactionMode === 'superstar')}
                        <Star color="var(--bg-color-1)" size="18"></Star>
                    {:else}
                        <Heart color="var(--timeline-reaction-liked-icon-color)" size="18"></Heart>
                    {/if}
                </button>
            {/if}

            {#if (item.reason === 'subscribed-post')}
                <button class="notification-icon notification-icon--post" onclick={handleClick}>
                    <Pencil color="var(--primary-color)" size="16"></Pencil>
                </button>
            {/if}
        </div>

        <div class="notification-column__content">
            <div class="notification-authors">
                {#each item.notifications as notification (notification)}
                    <div class="notification-author">
                        <Avatar href="/profile/{ notification.author.handle }" avatar={notification.author.avatar} handle={notification.author.handle} {_agent} profile={notification.author}></Avatar>
                    </div>
                {/each}
            </div>

            <h2 class="notifications-item__title">
                <span class="notifications-item__name">
                  <ProfileCardWrapper handle={item.notifications[0].author.handle} {_agent}>
                    <a class="notifications-item__link" href="/profile/{item.notifications[0].author.handle}">{item.notifications[0].author.displayName || item.notifications[0].author.handle}</a>
                  </ProfileCardWrapper>
                </span> {$_(getReasonText(item.notifications.length === 1 ? item.reason : item.reason + '_multiple'))}
            </h2>

            {#if (post)}
                <div class="notifications-item__content">
                    <p><a href="{'/profile/' + post.author.handle + '/post/' + post.uri.split('/').slice(-1)[0]}" onclick={handlePostClick}>{post.record.text}</a></p>
                </div>

                {#if (AppBskyEmbedImages.isView(post?.embed) && post?.embed)}
                    <div class="notifications-item-images">
                        <Images images={post.embed.images} blobs={post.record.embed.images} did={post.author.did}></Images>
                    </div>
                {/if}
            {:else}
                {#if item.notifications[0].reasonSubject.includes('app.bsky.feed.generator')}
                    <div class="notifications-item__feed">
                        <FeedEmbed {_agent} feedUri={item.notifications[0].reasonSubject}></FeedEmbed>
                    </div>
                {/if}
            {/if}
        </div>
    </div>
</article>

{#if isLikesOpen}
    <LikesModal uri={post.uri} onclose={() => {isLikesOpen = false}} {_agent}></LikesModal>
{/if}

{#if isRepostsOpen}
    <RepostsModal uri={post.uri} onclose={() => {isRepostsOpen = false}} {_agent}></RepostsModal>
{/if}

<style lang="postcss">
    .notification-authors {
        display: grid;
        grid-template-columns: repeat(auto-fill, 30px);
        gap: 4px;
        margin-bottom: 8px;
    }

    .notification-icon {
        width: 30px;
        height: 30px;
        border-radius: 50%;
        display: grid;
        place-content: center;
        transition: opacity .25s ease-in-out;
        position: relative;

        &::before {
            content: '';
            display: block;
            position: absolute;
            inset: 0;
            border-radius: 50%;
            background-color: var(--primary-color);
            opacity: .05;
            z-index: 1;
        }

        &--repost {
            &::before {
                background-color: var(--timeline-reaction-reposted-icon-color);
            }
        }

        &--like {
            &::before {
                background-color: var(--timeline-reaction-liked-icon-color);
            }
        }

        &:hover {
            opacity: .8;
        }
    }

    .notifications-item-images {
        margin-top: 8px;
    }
</style>