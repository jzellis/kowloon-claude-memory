---
name: feedback-sanitize-wrapper
description: "Always import sanitizeHtml from #methods/utils/sanitize.js in the server, never from 'sanitize-html' directly. The wrapper patches CVE-2026-44990 (xmp raw-text bypass) and the upstream package has no patched release yet."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

In the Kowloon server, never `import sanitizeHtml from "sanitize-html"` directly. Always go through `#methods/utils/sanitize.js`, which adds `'xmp'` to `nonTextTags` so disallowed `<xmp>` elements drop their contents instead of leaking raw script through.

**Why:** CVE-2026-44990 / GHSA-rpr9-rxv7-x643. sanitize-html ≤ 2.17.3 has a sanitizer bypass: its `ontext` handler special-cases `xmp` and `textarea` and dumps their raw contents into the output unescaped. A disallowed `<xmp><script>...</script></xmp>` payload becomes a real `<script>` tag in the sanitized output. No patched upstream release exists yet (`first_patched_version: null`). We mitigated in code in server commit `af51eeef` by routing all 8 call sites through the wrapper, verified against the GHSA PoC payloads. GitHub Dependabot alert #140 dismissed 2026-05-18 with reason `tolerable_risk`.

**How to apply:**
- Any new sanitize-html call site in the server must import from `#methods/utils/sanitize.js`. The signature is identical (it's a drop-in wrapper).
- `methods/utils/sanitize.js` always merges `'xmp'` into `nonTextTags`; callers can extend the list but can't remove it.
- If upstream sanitize-html ever ships a fix (`first_patched_version` populates on the GHSA), upgrade the package, drop the wrapper, and revert all 8 import paths back to `"sanitize-html"`.
- The frontend doesn't currently use sanitize-html. If it ever does, mirror this pattern there too.
- See [[project-security-hardening]] for the broader server XSS/CORS/SSRF hardening context.
