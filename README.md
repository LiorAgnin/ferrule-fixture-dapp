# Ferrule fixture dapp

Static page Ferrule's Patrol Android tests load in the in-app browser. No
framework, no CDN, no fonts, no analytics, no outbound requests. Android
WebView content is invisible to Patrol, so this page writes its state into
`document.title`. Ferrule already forwards that into the capsule
(`browser_capsule`) and the page record.

Live URL: https://lioragnin.github.io/ferrule-fixture-dapp/

Protocol and query parameters match Ferrule phase-11 spec §8:
https://github.com/LiorAgnin/ferrule/blob/main/docs/specs/phase-11-e2e/spec.md

## Title protocol

| `document.title` | Meaning |
| --- | --- |
| `ferrule:loading` | page parsed, provider not yet probed |
| `ferrule:ready` | `window.ethereum` present and `isFerrule` is true |
| `ferrule:no-provider` | no injected provider (flagged origin, or injection off) |
| `ferrule:connected:<0xaddr>` | `eth_requestAccounts` resolved |
| `ferrule:chain:<hex>` | `eth_chainId` result |
| `ferrule:signed:<first 10 hex of sig>` | `personal_sign` resolved |
| `ferrule:error:<code>` | provider rejected (EIP-1193 error code) |

The connected address is lowercase. The signed suffix is the first 10 hex
characters of the signature after `0x`. Error code is `err.code` when it is a
number, else `unknown`.

## Query parameters

Android tests do not tap inside the page. They load an action on navigation:

- `?auto=connect` runs `eth_requestAccounts`
- `?auto=chain` runs `eth_chainId`
- `?auto=sign` runs `eth_requestAccounts`, then `personal_sign` of the UTF-8
  message `ferrule-fixture` from the first account

Connect, Chain, and Sign buttons run the same three actions for a human. Every
title change is also appended to `#log`.

## Provider match

The page waits up to 3 seconds (poll every 100 ms) for Ferrule's injected
provider:

- `window.ethereum.isFerrule === true`
- an EIP-6963 announce whose `info.rdns` is `money.ferrule.app`

Those values are the ones Ferrule injects today from
`assets/browser/ferrule-provider.js` (`isFerrule` locked true, rdns
`money.ferrule.app`). The Dart loader is
`lib/data/browser/ferrule_provider_script.dart`. Desktop Chrome with no wallet
settles on `ferrule:no-provider` within 3 seconds.
