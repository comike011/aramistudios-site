# aramistudios-site

@.claude/fragments/workflow-transitions.md
@.claude/fragments/worktree-requirements.md
@.claude/fragments/app-repo-contract.md

## App

<!-- Every field is required. The /af-* commands stop if one is missing. -->

- **Stack:** Static website for aramistudios.com. Hand-written HTML, CSS and vanilla JS; no framework, no build step, no runtime dependencies.
- **Build:** `none` (the repo root is served as-is)
- **Test:** `npx --yes html-validate@9 "*.html" "privacy/*.html"`
- **Lint:** `npx --yes prettier@3 --check "*.html" "privacy/*.html"`
- **Setup:** `none` (preview locally with `python3 -m http.server 8000`)
- **Bundle id:** n/a (website; domain `aramistudios.com`)
- **GitHub:** `comike011/aramistudios-site`

## Conventions

- The approved design is `docs/mockup.html` ("Quiet Sky" with drifting pixel clouds). Build to it; changing its direction needs the owner's sign-off.
- Keep it light: a homepage plus `/privacy/`, each page self-contained with inline CSS (and JS where needed), Google Fonts as the only external request. No trackers or analytics unless asked.
- Every color is a CSS token with light and dark values. Animation respects `prefers-reduced-motion`.
- Copy rules: Arami Studios is a small app studio. Don't describe it as a game studio or as making apps for kids, and never name the owner's children. The name note may say it "started as a family story" and that *arami* means "little sky" in Guaraní.
- Contact email is `support@aramistudios.com`.
