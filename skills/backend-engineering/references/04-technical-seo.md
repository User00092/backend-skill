# Technical SEO and indexing checklist

> Extracted from `archive/backend-skills-original.md` (repository root). Original annotations, quoted source claims, and synthesis distinctions are preserved where included in the topic. Source descriptions and notes not embedded here remain in that archive.

**Source:** @joshtheaiguy — 2026-09-12 — https://www.instagram.com/reel/DdMjkxiyaJ/

**Source note:** The text attachment has only a brief caption. The actionable details below were read from **on-screen checklists** in the video, not from a speech transcript.

### On-screen items
- ✅ Build a `sitemap.xml`.
- ✅ Build a `robots.txt`.
- ✅ Meta titles.
- ✅ Meta descriptions.
- ✅ Canonical tags.
- ✅ Only one H1 tag per page (presented as a recommended rule in the clip).
- ✅ Fix header hierarchy.
- ✅ Add alt text to images.
- ✅ Improve Core Web Vitals.
- ✅ Internal links.
- ✅ Add schema markup.
- ✅ Compress images.
- ✅ Fix mobile responsiveness.
- ✅ Add an `og:image`.
- ✅ Build a backlink strategy.
- ✅ Clean up URL slugs.
- ✅ Verify Search Console.
- ❌ `noindex` tags.
- ❌ Broken links.

### Interpretation and cautions
- Check `noindex` on pages intended for public indexing; **do not** remove it indiscriminately from staging, private, filtered, or duplicate pages.
- A sitemap helps discovery but does not guarantee indexing. `robots.txt` controls crawler access; it is **not** a secrecy mechanism or a guaranteed de-indexing tool.
- A single H1 can be a useful editorial convention, but multiple H1 elements are not inherently an indexing prohibition.
- Alt text should convey meaningful image information, not keyword stuffing; decorative images can have empty alt text.
- Structured data must represent real page content and comply with search engine policies.
- Backlinks should be earned and relevant, not purchased or artificially manufactured.

### Engineering checklist
- [ ] Return indexable, canonical, accessible HTML for intended public pages.
- [ ] Generate valid `robots.txt` and XML sitemaps.
- [ ] Inspect `noindex`, canonical URLs, redirects, broken links, and HTTP status codes.
- [ ] Create descriptive metadata, headings, internal linking, and image text.
- [ ] Improve loading speed, mobile layout, structured data, and OG previews.
- [ ] Validate with actual crawl data and search console rather than assuming ranking improvements.
