# changelog-gen — Generate Human-Readable Changelog

## When to Use
DevRel uses this weekly to generate a changelog entry from recent git history, focused on user-facing changes.

## Inputs
- Git log since last changelog entry
- Existing CHANGELOG.md (to determine last entry date and format)

## Procedure

1. Read the last entry in CHANGELOG.md to determine the cutoff date
2. Run `git log --oneline --since="{cutoff}"` to collect all commits since then
3. Group commits by type using conventional commit prefixes:
   - **feat**: New features
   - **fix**: Bug fixes
   - **refactor**: Significant internal changes (only if user-facing impact)
   - **perf**: Performance improvements
   - **docs**: Documentation updates
4. Filter out purely internal changes (test-only, CI config, dev tooling) unless significant
5. Write each entry in human-readable language — translate commit messages into what the user experiences
6. Append the new entry to CHANGELOG.md:

```markdown
## [{version or date}] — {date}

### Added
- {user-facing description of new feature}

### Fixed
- {user-facing description of bug fix}

### Changed
- {user-facing description of significant change}

### Improved
- {performance or UX improvement}
```

7. Post summary to `#daily-standup`

## Output Format
New section appended to CHANGELOG.md following Keep a Changelog format. Summary posted to `#daily-standup`.

## Rules
- Write for users, not developers — "Enrichment results now include phone numbers" not "feat(enrichment): add phone field to response DTO"
- Skip commits that have zero user-facing impact (test refactors, lint fixes, CI tweaks)
- Include refactors only if they change behavior, performance, or API surface
- Never overwrite existing changelog entries — append only
- If no user-facing changes this week, note "No user-facing changes" rather than skipping
