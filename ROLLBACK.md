# Dark theme (beta) - rollback

The grayscale builder theme lives in `theme-darkweb.css` and is gated by a flag. With no flag set the site is unchanged.

**Turn it on:** open the site with `?theme=darkweb` (remembered in this browser via localStorage key `lead_theme`).

**Turn it off (any one of these):**

1. Visit the site with `?theme=default`. This clears `lead_theme` and the page loads in the original look.
2. Remove the theme from the code: delete `theme-darkweb.css`, and delete the `<link rel="stylesheet" href="theme-darkweb.css">` line plus the flag `<script>` right after it in the `<head>` of `index.html`. Those two blocks are the only edits to `index.html`.
3. Revert to the code as it was before the theme: `git revert` the theme commit(s), or `git checkout pre-darkweb-theme -- index.html` and delete `theme-darkweb.css`. The tag `pre-darkweb-theme` marks the last commit before any theme work.

Users who previously opened `?theme=darkweb` keep `lead_theme=darkweb` in their browser until they visit `?theme=default`. After option 2 or 3 that value is ignored (nothing reads it).
