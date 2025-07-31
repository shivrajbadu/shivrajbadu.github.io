# GEMINI CLI Custom Instructions

## Git Automation Workflow

When I use any of these trigger phrases, automatically execute the complete git workflow:

**Trigger Phrases:**
- "commit and push"
- "save changes" 
- "deploy"
- "update repo"
- "save work"
- "push to github"

**Automated Steps:**
1. Run `git status` to check current state
2. Execute `git add .` to stage all changes
3. Generate meaningful commit message based on file changes
4. Run `git commit -m "[generated message]"`
5. Execute `git push` to current branch

## Commit Message Convention

Follow conventional commit format:
- `feat: description` - New features
- `fix: description` - Bug fixes  
- `docs: description` - Documentation changes
- `style: description` - Formatting/styling
- `refactor: description` - Code restructuring
- `chore: description` - Maintenance tasks
- `update: description` - General updates

## Smart Message Generation

Analyze changed files to create specific commit messages:
- Blog posts → `docs: add [topic] blog post` 
- Code files → `feat: implement [feature]` or `fix: resolve [issue]`
- Config files → `chore: update [config type]`
- Multiple types → `update: [brief summary of changes]`

## Default Behavior

- Always check git status first
- Stage ALL unstaged changes (`git add .`)
- Push to current active branch
- Handle any merge conflicts or push errors gracefully
- Provide clear feedback on each step

## Example Interactions

**Me:** "commit and push these blog changes"
**You:** 
```bash
git status
git add .
git commit -m "docs: add Rails 8 modern development guide"
git push
```

**Me:** "save work"  
**You:** Automatically execute full workflow with appropriate commit message

This eliminates the need for multiple back-and-forth messages and manual git commands.
