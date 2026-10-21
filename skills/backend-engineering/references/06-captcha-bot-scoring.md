# CAPTCHA, risk scoring, and false positives

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @krishnachaytanyaa — 2026-08-19 — https://www.instagram.com/reel/DcOPmONtnQB/

### Lessons extracted
- CAPTCHA/bot defenses can consider session/cookie context, browser attributes, network reputation, and interaction signals; implementation varies by vendor/product.
- Bot detection is probabilistic and can generate both false positives (legitimate users challenged) and false negatives (automation accepted).
- The source describes reCAPTCHA v3 as producing scores from **0.0 to 1.0** and cites **0.5** as a default threshold example.

### Important qualification
The video blends concepts from multiple reCAPTCHA generations. **reCAPTCHA v3 is score-based and does not itself automatically show image challenges to users below 0.5**; an application may choose further verification or other actions based on the score. Avoid treating claimed mouse-tracking behavior or a universal threshold as an established public specification for every CAPTCHA system.

### Engineering checklist
- [ ] Validate verification tokens on the server, including expected site key, hostname/action, freshness, and other vendor-required checks.
- [ ] Tune risk thresholds using actual abuse and false-positive rates.
- [ ] Provide accessible alternatives and recovery routes for legitimate users.
- [ ] Layer anti-abuse controls: rate limits, replay prevention, monitoring, and account/session rules as appropriate.
- [ ] Avoid assuming a checkbox click proves human identity or that a score proves malicious intent.
