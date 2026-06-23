# How to publish this portfolio

This folder is a ready-to-push GitHub **profile README**. When pushed to a public repo named exactly `SeeTouch/SeeTouch`, the `README.md` shows automatically at the top of your profile page (https://github.com/SeeTouch).

## One-time publish (you are already logged in as `SeeTouch` via `gh`)

```bash
cd /Users/seetouch/mation/portfolio
gh repo create SeeTouch/SeeTouch --public --source=. --remote=origin --push
```

That creates the public repo and pushes everything. Done — open https://github.com/SeeTouch to see it.

## Updating later

```bash
cd /Users/seetouch/mation/portfolio
git add -A && git commit -m "Update portfolio" && git push
```

## Notes

- The Mermaid timeline and shields.io badges render natively on GitHub — no extra setup.
- Case-study links are relative, so they work on both the profile page and inside the repo.
- Repositories stay private; only these written case studies are public.
- To add screenshots: drop images into `assets/`, then reference them in a case study with `![alt](../assets/file.png)`. Several projects have real screenshots ready (see each case study's "Screenshots" section).

## Optional polish

- Add a `LICENSE` or keep it as-is (a profile repo needs none).
- Pin a couple of public repos to your profile if you ever open-source one.
- Custom domain / GitHub Pages site can reuse the same case-study content later if you want a branded URL.
