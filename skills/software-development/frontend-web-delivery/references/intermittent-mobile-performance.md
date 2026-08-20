# Diagnosing intermittent mobile slowness on static/SPAs

Use this when a site is fast in desktop checks or Lighthouse but a real phone intermittently loads slowly or appears not to load.

## Diagnostic sequence

1. **Separate origin health from client startup**
   - Probe apex and canonical hosts repeatedly with mobile user-agent timing fields: DNS, connect, TLS, TTFB, total, redirects, effective URL.
   - Check HTML, hashed JS/CSS, fonts, and critical images independently.
   - Run a throttled mobile Lighthouse test, but do not treat a high synthetic score as proof that real carrier paths are healthy.
2. **Inspect the raw HTML shell**
   - If `<body>` contains only an empty application root, the page has no useful fallback before JavaScript downloads and executes.
   - A delayed or failed bundle can therefore look like a totally dead page even when the origin itself is healthy.
3. **Audit cache headers by resource class**
   - HTML can use revalidation.
   - Content-hashed assets should normally use `Cache-Control: public, max-age=31536000, immutable`.
   - `max-age=0, must-revalidate` on hashed JS/CSS forces a network round trip on repeat visits and amplifies intermittent mobile connectivity.
4. **Count redirect and connection cost**
   - Apex-to-www or www-to-apex adds DNS/TLS/request work. On a weak carrier connection this can cost seconds.
   - Ensure canonical, Open Graph, sitemap, public links, and the serving host all use the same primary hostname.
5. **Check mobile-path factors**
   - Compare Wi-Fi and cellular behavior, operator, browser, device, Private Relay/VPN, DNS filtering, IPv4/IPv6 availability, and Vercel/CDN edge routing.
   - A-only hosting can still work through NAT64, but carrier-specific failures warrant testing from that carrier rather than assuming global availability.
6. **Inspect rendering cost separately from loading cost**
   - Inventory large bundles, external font requests, fixed/backdrop-blur elements, continuous animations, canvases/WebGL, and `requestAnimationFrame` loops.
   - Loading delay, blank startup, and post-load jank are different failure classes and should not be conflated.
7. **Add real-user evidence**
   - Capture Web Vitals and failures by device, browser, network type, hostname, geography, and release.
   - Ask for an exact timestamp and whether the failure occurred on Wi-Fi or cellular so CDN/provider logs can be correlated.

## Interpretation pattern

When repeated origin probes are fast and Lighthouse is strong, but raw HTML is empty and hashed assets require revalidation, report both facts: the server is currently healthy, while the delivery architecture remains fragile on intermittent mobile networks. Do not dismiss the user's real-device report because a synthetic score is high.

## Preferred remediation order

1. Give hashed assets long-lived immutable caching.
2. Prerender or SSR meaningful above-the-fold content so the first paint does not depend entirely on JavaScript.
3. Remove avoidable hostname redirects and make metadata hostname-consistent.
4. Reduce external critical dependencies and expensive mobile visual effects.
5. Validate with real-user monitoring and tests on the affected carrier/device.