# Contributing to Ceph Cluster Setup Guide

Thank you for your interest in contributing! This guide explains how to improve the documentation.

## How to Contribute

### Report Issues

Found an error, typo, or unclear instruction?

1. Check the [Issues](../../issues) page to see if it's already reported
2. If not, [create a new issue](../../issues/new) with:
   - Clear title describing the problem
   - Description of the issue
   - Which phase/section it affects
   - Your Ceph/OS setup (for context)

### Improve Documentation

Want to enhance the guides?

1. **Fork** the repository
2. **Create a branch** for your changes: `git checkout -b improve/phase-3-bootstrap`
3. **Edit** the markdown files in `/docs/`
4. **Test** your instructions (if possible)
5. **Commit** with a clear message: `git commit -m "Clarify pool replica configuration"`
6. **Push** your branch: `git push origin improve/phase-3-bootstrap`
7. **Open a Pull Request** with:
   - Description of changes
   - Why the change improves the guide
   - Any testing you performed

### Add New Content

Want to add a new phase or guide?

Examples of valuable additions:
- Multi-node cluster setup
- Ceph troubleshooting guide
- Performance tuning
- Security hardening
- Integration guides (Kubernetes, OpenStack, Proxmox)
- CLI command reference

**Process:**
1. Fork and create a branch
2. Create new `.md` file in `/docs/` with clear naming: `09-NEW-TOPIC.md`
3. Update `README.md` table of contents if needed
4. Ensure consistent formatting with existing guides
5. Submit PR with detailed description

## Guidelines

### Writing Style

- **Clear and concise** — avoid jargon, explain new terms
- **Step-by-step** — break complex tasks into numbered steps
- **Practical** — include real commands and expected output
- **Actionable** — readers should be able to follow without external resources
- **Explanatory** — include "why" not just "how" (use blockquotes for this)

### Code Blocks

- Use markdown fenced code blocks with language specification
- Include comments for non-obvious commands
- Show expected output when helpful

**Example:**
```bash
# Create a 5GB RBD image
sudo rbd create rbd-pool/my-image --size 5G

# Verify creation
sudo rbd info rbd-pool/my-image
```

### Command Output

When showing command output:
- Use separate code block with output language (if supported)
- Truncate long output to relevant portions
- Explain key lines when complex

**Example:**
```
mds.cephfs.ceph-ssd.fnztka  ceph-ssd     running  1m ago  2m  40.0M  16.0G  18.2.4
```

### Links

- Use relative links for docs: `[Phase 5](./05-ADD-OSD.md)`
- Use full URLs for external resources: `[Ceph Docs](https://docs.ceph.com)`
- Link to related guides when relevant

### Images

If adding diagrams/screenshots:
1. Save in high quality
2. Add descriptive alt text
3. Keep file size reasonable
4. Use `![description](path-to-image)` format

## Testing

Before submitting:

1. **Read through** your changes for clarity and typos
2. **Check formatting** — headings, code blocks, lists are consistent
3. **Verify links** — relative and external links work
4. **Test commands** (if possible in your environment)
5. **Review** the style matches existing documentation

## Branch Naming

Use descriptive branch names:

- `docs/phase-1-update` — documentation updates
- `fix/typo-in-phase-3` — bug fixes
- `feature/kubernetes-guide` — new features
- `improve/better-explanations` — improvements

## Commit Messages

Write clear commit messages:

```
Short description (max 50 chars)

Longer explanation (if needed, max 72 chars per line)
- What changed
- Why it changed
- Any relevant context
```

Example:
```
Fix capitalization in Phase 2

The cephadm binary documentation had inconsistent 
capitalization. Standardized to match official Ceph docs.
```

## Pull Request Process

1. Describe what changed and why
2. Reference related issues: `Fixes #123`
3. Explain any impact on other sections
4. Request review if needed
5. Be open to feedback and discussion

## Question or Need Help?

- **Create a Discussion** — for questions about the guide
- **Check existing issues** — someone may have already asked
- **Comment on related PRs** — ask for clarification if needed

## Recognition

Contributors are recognized for their valuable work. Your GitHub username will be credited for significant contributions.

---

Thank you for making this guide better! 🙌
