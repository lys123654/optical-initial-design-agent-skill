# GitHub Publish Checklist

Before publishing:

- Keep `SKILL.md`, `README.md`, `templates/`, `docs/`, `scripts/README.md`, and `examples/`.
- Do not publish private Zemax license paths if they reveal sensitive environment details.
- Do not publish proprietary lens files or private competition attachments.
- If publishing real patent examples, keep source URLs and note that patent prescriptions may not be final products.
- Add a license if the repository will be public.
- Test installation from a clean folder.

Suggested commands:

```powershell
git init
git add .
git commit -m "Add optical initial design agent skill"
git branch -M main
git remote add origin https://github.com/<your-name>/<repo>.git
git push -u origin main
```

If GitHub CLI is authenticated:

```powershell
gh repo create <your-name>/<repo> --public --source . --remote origin --push
```

