# Music Monetization Map

`music-distribution` is not limited to music distributors. The database now treats **any route by which music or music-related value can generate money** as a monetization channel.

## Layers

1. **Distribution** — get recordings into DSPs and stores.
2. **Direct-to-fan** — sell downloads, physical releases, merch and bundles directly.
3. **Streaming / video** — monetize listening and viewing.
4. **UGC / rights** — Content ID, creator music, neighboring/recording-rights monetization.
5. **Physical** — CDs, vinyl, cassettes, merchandise.
6. **Live** — tickets, streams, listening parties and events.
7. **Fan funding** — tips, memberships, subscriptions and contributions.
8. **Commerce infrastructure** — own site + payment processor.
9. **Licensing** — sync, commercial use, samples, commissions and B2B licensing.
10. **Rights administration** — publishing, collection and royalty administration.

## Current additional channels

| Channel | Main money route | Direct sale | Physical | Streaming | Rights | Key evidence |
|---|---|---:|---:|---:|---:|---|
| Daiki Sound | distribution / CD / karaoke | No | Yes | Yes | Yes | `daiki-sound-service` |
| Bandcamp | digital / physical / merch | Yes | Yes | No | Limited | `bandcamp-fees` |
| SoundCloud | streaming / distribution / fan support | Yes | Yes | Yes | Yes | `soundcloud-artist` |
| YouTube | ads / Premium / fan funding / rights | No | No | Yes | Yes | `youtube-ypp`, `youtube-content-id` |
| Personal site + Stripe | downloads / goods / subscriptions / tickets | Yes | Yes | No | Depends on implementation | `stripe-payment-links` |

## Why these should not be forced into `distributors.json`

A distributor and a storefront are different objects.

- DistroKid can deliver a recording to stores.
- Bandcamp can sell the recording directly to a fan.
- YouTube can monetize viewing, fan funding and rights claims.
- Stripe can process the payment on an artist-owned storefront.
- Daiki Sound can combine digital distribution with physical retail and karaoke.

Therefore the knowledge graph should model:

```text
artist
  ├── recording
  ├── rights
  ├── audience
  └── products
        ↓
monetization_channel
        ↓
revenue_event
        ↓
royalty / fee / tax / payout
```

## Query examples

The eventual RAG/API should answer questions such as:

- 「CDを100枚売るならどこがいい？」
- 「Bandcampと自サイト+Stripeはどっちが儲かる？」
- 「サブスクより直接販売を優先したい」
- 「TikTokで発見→YouTube→Bandcampで購入、という導線を作りたい」
- 「配信・CD・カラオケを全部まとめたい」
- 「ファンから月額で支援してもらいたい」
- 「自分のサイトで音源・Tシャツ・チケットを全部売りたい」
- 「Content IDまで含めて一番取りこぼしが少ない構成は？」

## Important semantic rule

Do not compare `100%` across systems as if it were the same metric.

- distributor royalty share
- direct-store revenue share
- payment-processing fee
- advertising revenue share
- publishing royalty
- master recording royalty
- physical product margin

are separate economic quantities.

The database therefore stores the **revenue mechanism and fee basis**, not only a single `artist_share` number.

## Evidence policy

Every volatile commercial claim should point to an atomic evidence record with:

- `entity_id`
- `claim`
- `source_url`
- `source_type`
- `checked_at`
- `confidence`

Primary sources are preferred. If a fee or eligibility condition changes, update the evidence record rather than silently changing the interpretation.
