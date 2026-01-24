# Software Engineering Research Patterns

## Library/Package Selection

### Essential Checks

1. **Maintenance health**
   - Last commit date (< 6 months ideal)
   - Open issues vs closed ratio
   - Response time on issues
   - Bus factor (single maintainer = risk)

2. **Adoption signals**
   - GitHub stars (trend matters more than absolute)
   - npm/PyPI weekly downloads
   - Used by notable projects (check dependents)

3. **Quality indicators**
   - Test coverage mentioned/visible
   - TypeScript types (for JS ecosystem)
   - Documentation quality and examples
   - Breaking change history in changelog

### Search Patterns

```
"{library} vs {alternative} {year}"
"{library} production experience"
"{library} issues performance"
"site:github.com {library} issues"
"{library} migration from {current}"
```

## API Integration Research

### Documentation Hierarchy

1. Official API reference (authoritative)
2. Official SDKs/client libraries
3. OpenAPI/Swagger specs if available
4. Official examples/tutorials
5. Community wrappers (verify maintenance)

### Critical Questions

- Rate limits and quotas?
- Authentication method (API key, OAuth, JWT)?
- Webhook support for real-time updates?
- Sandbox/test environment available?
- Breaking change policy and versioning?
- Error response format and codes?

### Red Flags

- Docs reference deprecated endpoints
- No versioning strategy documented
- Examples use outdated SDK versions
- No status page or uptime guarantees

## Architecture Research

### Finding Reference Implementations

```
"site:github.com {pattern} example"
"{company} engineering blog {topic}"
"{pattern} at scale"
"{framework} {pattern} implementation"
```

### Evaluating Architecture Decisions

Consider:
- What scale was this designed for?
- What tradeoffs did they accept?
- What would they do differently? (look for retrospectives)
- Does this match our constraints?

## Debugging Research

### Search Strategy

1. **Exact error message** (in quotes)
2. **Error + library + version**
3. **GitHub issues for the library**
4. **Stack trace key frames**

### Source Priority for Debugging

1. GitHub issues (often has maintainer insight)
2. Stack Overflow (check answer dates!)
3. Library changelog (was this fixed?)
4. Source code (ultimate truth)

### When to Stop Searching, Start Experimenting

- Same solutions appearing repeatedly
- Solutions are for different versions
- Error is environment-specific
- Need to isolate with minimal reproduction

## Dependency Research

### Security Checks

- `npm audit` / `pip-audit` / `cargo audit` output
- Known CVEs for the version
- Dependency tree depth (transitive risk)
- Last security patch timeline

### License Compatibility

Quick reference for common licenses:
- MIT, Apache 2.0, BSD: Generally permissive, check attribution requirements
- GPL: Copyleft, may require source disclosure
- LGPL: Copyleft for library modifications only
- Proprietary: Check terms carefully

## Version/Upgrade Research

### Upgrade Impact Assessment

1. Read CHANGELOG/MIGRATION guide first
2. Search: `"{library} upgrade {old_version} to {new_version}"`
3. Check GitHub issues tagged "migration" or "upgrade"
4. Look for codemods or migration scripts
5. Identify breaking changes affecting your usage

### Signals for "Wait vs Upgrade Now"

**Upgrade soon**: Security fixes, bug affecting you, needed features
**Wait**: Major version just released (let others find bugs), no pressing need
