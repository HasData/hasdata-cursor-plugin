---
name: creator-shortlist
description: Vet creators on TikTok, Instagram and YouTube, and state what each engagement number costs to get, since Instagram keeps likes, comments and timestamps on a separate per-post call rather than in the feed. Use for influencer shortlists, partnership due diligence, creator discovery and competitor channel research.
---

# Creator shortlist across platforms

The three platforms return very different payloads, and one of them cannot answer the question people usually ask. Say so early rather than inventing a number.

## What each platform can tell you

TikTok reports engagement per post. A post carries `likes`, `comments`, `shares`, `plays`, `reposts` and `collects`, with `createTime` as an ISO timestamp, so both recency and engagement are directly answerable.

YouTube reports views and likes. In the search and channel payloads `views` is the number and `viewsOriginal` is the printed string. The video call reverses it, where `views` reads `"482,284 views"` and `extractedViews` holds `482284`, matching `extractedLikes`. Read the type rather than the name and sort on whichever one is numeric.

Instagram splits them across two calls. The feed from `hasdata_instagram_posts_getInstagramPosts` carries `id`, `shortcode`, `caption`, `type`, `url` and owner fields, with no like count, no comment count and no timestamp. The numbers live on `hasdata_instagram_comments_getInstagramComments`, which takes a post URL and returns a `post` object with `likesCount`, `commentsCount` and `timestamp` beside the comment thread. So ranking an Instagram feed by engagement costs one extra call per post. Say that cost out loud before making twelve of them, and never rank by caption length or feed position to avoid it.

## Gather

TikTok is `hasdata_tiktok_profile_getTikTokProfile` for the account totals, then `hasdata_tiktok_posts_getTikTokPosts` for the feed, both on `handle`. To find creators rather than check one, `hasdata_tiktok_search_getTikTokSearch` with `type` set to `user`.

Instagram is `hasdata_instagram_profile_getInstagramProfile` on `handle`. The profile already embeds `latestPosts`, and the first page of `hasdata_instagram_posts_getInstagramPosts` repeats those same twelve, so the profile call is the cheaper start. Add `hasdata_instagram_comments_getInstagramComments` per post only when the engagement numbers are actually needed.

YouTube is `hasdata_youtube_channel_getYoutubeChannel`, where `channelId` also accepts an `@handle`, with `tab` set to `videos` for the upload history. To find channels, `hasdata_youtube_search_getYoutubeSearchResults` with `videoType` set to `channel`.

## Failures that look like answers

A TikTok handle that does not exist returns 200 with `requestMetadata.status` of `ok` and an `error` string where the profile should be. Test for the object you came for, or a missing account gets reported as an account with nothing to say. A malformed handle fails differently, with a 422 on `handle`, so the two need different messages.

A private Instagram account answers with `private: true` and little else, so branch on that flag before reading anything. A handle that does not resolve returns `isError` with a 400, and the same flag also carries a 401 for a bad key, so read the error text rather than only the flag.

A YouTube video id that does not resolve still answers 200 with no `transcript` and no `availableTranscripts` keys at all, so check the key exists before summarising.

## Do not over-read the optional fields

On TikTok, `hashtags` and `mentions` appear only on videos that carry them, and the rate follows the account rather than the tool. Sampled feeds ran from no tagged posts at all to every post tagged, so one account says nothing about the next, and an absent array means this account does not tag. Parse the `description` when tags matter.

The same holds on Instagram, where `mentions` is common and `hashtags` less so. Parsing the caption finds them either way.

The YouTube `publishedDate` is a relative string such as `4 years ago` and cannot be turned into a date. A time window belongs in the `date` filter on the search rather than in post-processing.

YouTube search splits into `videoResults`, `inlineShortsResults`, `playlistResults`, `adsResults` and `sponsoredResults`. Folding the last two into the organic set reports paid placements as reach.

## Report

One row per creator per platform with the follower or subscriber count, the posting cadence, and median engagement where the platform supplies it. For Instagram, either fill the engagement cell from a per-post comments call or leave it empty and say which, since the two cost very different amounts. A shortlist built on three platforms where one of them is guesswork is worse than a shortlist that admits the gap.
