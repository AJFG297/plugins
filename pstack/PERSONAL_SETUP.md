# Maintain the personal pstack branch

The `personal-pstack` branch is the source for Aidan's installed pstack skills. It keeps a small set of personal changes on top of [`cursor/plugins`](https://github.com/cursor/plugins) while those changes still benefit from upstream updates.

Do not commit model choices to this repository. Keep them in `~/.agents/pstack-models.md`.

## Know which copy to edit

- Edit the clone at `~/Documents/Personal/repos/pstack-plugins`.
- Push changes to `AJFG297/plugins`, branch `personal-pstack`.
- Treat `~/.agents/skills` as installed output. Do not edit skills there.
- Keep provider access, model choices, and effort levels in `~/.agents/pstack-models.md`.

## Change a skill

1. Switch to the personal branch and update it.

   ```bash
   cd ~/Documents/Personal/repos/pstack-plugins
   git switch personal-pstack
   git pull --ff-only origin personal-pstack
   ```

2. Edit files under `pstack/`, then review and commit the change.

   ```bash
   git diff -- pstack
   git add pstack
   git commit
   git push origin personal-pstack
   ```

3. Update the installed skills from the pushed branch.

   ```bash
   npx skills update --global --yes
   ```

The Skills CLI lockfile records `AJFG297/plugins`, ref `personal-pstack`, for the installed pstack skills.

## Install on another machine

Install pstack from the personal branch, then create the machine's private model configuration.

```bash
npx skills add https://github.com/AJFG297/plugins/tree/personal-pstack/pstack \
  --global \
  --agent codex \
  --agent cursor \
  --agent claude-code \
  --skill '*' \
  --yes
```

Run `/setup-pstack` to create `~/.agents/pstack-models.md`. Do not copy that file into this repository.

## Merge upstream changes

Merge upstream in this clone so personal changes stay visible as normal commits.

```bash
cd ~/Documents/Personal/repos/pstack-plugins
git switch personal-pstack
git fetch upstream
git merge upstream/main
```

Resolve conflicts in `pstack/`, review the combined diff, and test the affected skills. Then push and update the installed copies.

```bash
git push origin personal-pstack
npx skills update --global --yes
```

Do not merge `personal-pstack` into the fork's `main` branch. Keeping the changes on their own branch makes upstream comparison and updates easier.

## Leave the fork when it stops helping

This fork is useful while most of the skill set still comes from pstack. Move the personalized skills into a standalone repository when upstream merges cost more than the updates are worth, or when the skills no longer share pstack's structure and assumptions.

At that point:

1. Copy only the skills that remain useful into a new repository.
2. Keep `~/.agents/pstack-models.md` as local configuration, unless the new skills replace that contract.
3. Install from the new repository so the Skills CLI lockfile stops tracking `AJFG297/plugins`.
4. Archive or delete `personal-pstack` after the new installation works.
