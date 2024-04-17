# Proposal: Manifest trial_tokens

**Summary**

The manifest key to include origin trial tokens.

**Document Metadata**

**Author:** @bershanskyi

**Sponsoring Browser:** Chromium

**Contributors:** @rdcronin, @xeenon, @patrickkettner, @dotproto

**Created:** 2024-04-19

**Related Issues:**
 - https://github.com/w3c/webextensions/issues/454
 - https://issues.chromium.org/issues/40282364

## Motivation

### Objective

This new manifest key empowers developers to use trial tokens in WebExtension
service worker contexts, including the background service workers, which was
not possible before. Also, this manifest key simplifies injection of tokens
into all other contexts with the extension's origin.

#### Use Cases

Greater availability and beter ergonomics of Origin and Deprecation tokens.

##### Participation in Origin and Deprecation Trials for Open Web APIs

Enroll extension contexts, in particular extension background service workers
in origin and deprecation trials. Origin Trials allow developers to
explore new APIs and behaviors before general availability and to extend
legacy behaviors and deprecated APIs.

##### Creation of Origin and Deprecation Trials for WebExtension APIs

As of writing, WebExtension system does not have a convenient way of shipping
changes to WebExtension APIs. In leu of Origin Trials framework browsers resort
to workarounds (or their combination):
 - make the change backwards-compatiable, if possible, with perpetually
   deprecated legacy behavior
 - launch a change to pre-release Canary/Nightly and Beta channels and
   and ask for developer feedback prior to launch to Stable. Some issues slip
   through this testing and get fossilized in the final API.
 - limit the change via allow- and block-lists specified at in browser source
   code
 - launch an enirely different API under a new name
 - Chromium explicitly introduced breaking Manifest Version transition (first
   from Manifest Version 1 to Manifest Version 2, and now from Manifest Version
   2 to Manifest Version 3). This is taxing on developers since developers may
   have to maintain code paths for both API behaviors for the entire duration of
   migration.

The new manifest key `trial_tokens` can enable per-change opt-in and this may
speedup API iteration.

See also [Chromium source code](https://github.com/chromium/chromium/blob/main/chrome/common/extensions/api/_api_features.json)

### Known Consumers

None at the moment.

## Specification

### Schema

This is a manifest change with no programmable API.

This proposal extends manifest with optional `trial_tokens` member which
contains an array of strings which represent the stringified trial tokens.

Parsed and recognized tokens should be exposed via existing
[`runtime.getManifest()`](https://developer.mozilla.org/docs/Mozilla/Add-ons/WebExtensions/API/runtime/getManifest)
API in `origin_trials`. Invalid or unrecognized tokens should be excluded
from the returned manifest. This will enable extensions to detect recognized
tokens and adjust accordingly.

### Behavior

#### Manifest parsing

Upon manifest parsing, browser should parse `trial_tokens` like so:
 - `trial_tokens` key must never cause installation errors, only benign
   warnings
 - browsers can limit the total number of parsed and/or accepted tokens
 - browsers can limit the individual and total lengths of parsed and/or accepted tokens
 - browsers can validate token structure, content, and signature at parse time

#### Manifest parsing algorithm
 1. if `trial_tokens` is not present, return
 2. if `trial_tokens` is not an array, log a benign warning and return
 3. for every element `token` of `trial_tokens`:
   3.1. (optionally) if the maximum number of accepted tokens is reached,
        skip all other tokens
   3.2. if `token` is not a string, skip it and (optionally) log a benign
        warning
   3.2. if `token` is an empty string, skip it and (optionally) log a benign
        warning
   3.3. (optionally) if `token` is too long to be parsed, skip it and
        (optionally) log a benign warning
   3.4. if token has been encountered before, skip it and (optionally) log
        a benign warning
   3.5. (optionally) attempt to parse the token and validate it. If validation
        fails, skip the token and (optionally) log a benign warning.
        Note: browsers may disregard token expiration errors, if they are not
        sure about the corrctness of system timestamp at the time of token
        validation
   3.6. append token to the collection of accepted tokens

### New Permissions

No new permissions.

### Manifest File Changes

Add new manifest member called `trial_tokens` which is a non-empty array of
non-empty strings representing trial tokens.

## Security and Privacy

### Exposed Sensitive Data

This new key does not expose any sensitive data by itself, but it may activate
security- and privacy-critical code paths in the browser itself.

### Abuse Mitigations

Trial tokens system already incorporate some degree of abuse mitigations:
 1. Browser vendor signs every token for every origin and feature.
 2. Browser vendor audits every trial feature similarly to a regular feature.
 3. Browser vendor can invalidate any token individually or all tokens for
    a particular feature remotely.

### Additional Security Considerations

This new manifest key contains values which will be inserted into headers of
local HTTP-like responses. Browsers must ensure that the provided tokens can
not escape header serealization. Browsers also must ensure that tokens do not
cause headers to exceed implementation-specific limits so that response is
processed in  different way (e.g., adding a large number of tokens must not
cuase removal of security headers like CSP).

## Alternatives

### Existing Workarounds

### Open Web API

Document contexts can activate tokens by
[embedding them in DOM](<meta http-equiv="origin-trial" content="TOKEN_GOES_HERE">)
`<meta>` tag, both statically and dynamically. Extension service workers can
spawn documents and embed tokens and proxy calls to trial features via the said
document.

## Implementation Notes

Please see "Manifest parsing algorithm" section.

## Future Work

Highlight any planned or prospective future work.
