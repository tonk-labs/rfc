# Sandbox

## Problem

1. Currently all tonks and the runtime share [same origin](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) that implies that any malicious tonk (js code) could gain unlimited (sudo) access to everything.

1. It also implies that bad code can block everything unintentionally and runtime has no way of recovering from it.

1. Bad user content can lead to browsers blocking the origin as malware taking down the whole system. IPFS gateways had to deal with that a lot and it can take weeks to resolve.
   > ℹ️ Surfacing CID in the origins was ultimately a solution for this problem.

## Proposal

For the very least we can move system layer into an origin that is separate from the app layer. This can ensure that account / profile keypairs are inaccessible by the apps (tonks) and insead each app is delegated access to what it needs.

> ℹ️ This does not attempt to isolate apps from each other, but rather ensure that provenance is tracable and user can revoke access as needed. It also means app can always request for more capabilities as it needs.

System layer can load individual apps (tonks) in a [sandboxed iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/iframe#sandbox)s in suborigin restricting it's access to system resources. If suborigins are unique to each app (tonk) each one will end up being isolated from others.

### Complication

There is a significant complication with this design because parent Service Worker will not respond to requests from sub-origins. Overcoming this limitation was [demonstrated in lunet](https://gozala.io/lunet.html) experiment, by pre-installed service worker in the sub-origin and arranging a [MessagePort](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort) to system origin so it could access system resources via RPC.
