# Dark theme - rollback

The dark grayscale builder theme is the default look. It lives in `theme-darkweb.css` and is switched on by the `theme-darkweb` class on the `<html>` tag in `index.html`. The report preview and print output are not affected by it.

**Go back to the original light builder (any one of these):**

1. Remove ` class="theme-darkweb"` from the `<html lang="en">` tag in `index.html`. The theme file then does nothing and the original look returns.
2. Revert the commit that made the theme the default: `git revert <commit>`.
3. Restore the pre-theme code: `git checkout pre-darkweb-theme -- index.html` and delete `theme-darkweb.css`. The tag `pre-darkweb-theme` marks the last commit before any theme work.
