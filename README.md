# ads_test

THIS IS PURELY FOR TESTING ADS. Nothing here is production code — the ad units
are public sample units, and the Prebid bundle is a `not-for-prod` build.

## The four steps

Generally speaking, to set up ads on a webpage you do the following:

1. **Inject the script into the page.** — `loadScript()`
2. **Define spaces/containers where ads live.** — the `.ad` divs + `SLOTS`
3. **Request ads using the script/api.** — `defineSlot()` / `requestBids()`
4. **Display the ads.** — `display()` / `renderAd()`

Both paths in `index.html` follow those steps in that order. The thing that
makes ads hard to debug is that a failure at any step looks identical from the
outside: an empty box. So every step here logs, and any slot that ends up
unfilled says why in place of the creative.

## Running it

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Slots

| id | placement | size |
| --- | --- | --- |
| `banner-ad` | sidebar, sticky rail | 300×250 |
| `ad-sticky` | fixed bottom bar, all viewports | 320×50 |

Both reserve their height up front so filling them doesn't shift the layout.

## Feature flags

Read from the URL query. Accepts `1/0`, `true/false`, `yes/no`, `on/off`.
Precedence is **query → preset `window.APP_CONFIG` → default**.

| URL | GAM | Prebid |
| --- | --- | --- |
| `/` | on | off |
| `?showGAM=0` | off | off |
| `?showPBJS=1` | off | on |
| `?showGAM=1&showPBJS=1` | on | on |

GAM and Prebid are alternate paths, so asking for one turns the other off.
Name both explicitly for the header-bidding case. An unrecognised value logs a
warning and falls through to the default rather than reading as `false`.

Add a flag by adding one line to `FLAG_DEFAULTS`.

## GAM path (`?showGAM=1`)

Google Publisher Tag against the public sample unit
`/6355419/Travel/Europe/France/Paris`. `slotRenderEnded` logs `isEmpty`,
`size`, `creativeId`, and `lineItemId` per slot.

That sample unit only serves a few sizes — 300×250 and 320×50 fill; 300×600
comes back empty. An empty response there means the ad unit has no matching
creative, not that the wiring is broken.

## Prebid path (`?showPBJS=1`)

Runs an auction and renders the winning bid **directly into the slot** with
`pbjs.renderAd()`. It does not hand off to GAM.

That is deliberate. Normally Prebid sets `hb_pb` targeting and GAM picks the
winner, but the sample ad unit has no Prebid line items — so a bid could win
the auction and still never render, and the slot would look broken while
being wired correctly. Rendering standalone removes GAM from the picture.

Demand is AppNexus test placement `13144370`.

**Known issue:** a direct probe of that placement returned `nobid`. The probe
had no referrer or cookies, so it isn't conclusive, but expect the possibility
of "No bid" in the slots. The 320×50 sticky will almost certainly no-bid — it's
a 300×250 test placement. If you need guaranteed fill, swap
`PBJS_TEST_PLACEMENT`, or stub a bid adapter that returns a synthetic bid.

## Not implemented

No consent or cookie handling of any kind — no CMP, no TCF/GPP, no
`setPrivacySettings()`, no `disableInitialLoad()`. GPT is requested as soon as
the flag says so. Fine for a local test page, not fine anywhere real.
