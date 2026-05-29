# clj-debug Changelog

All notable changes to this skill will be documented in this file.

## [1.0.0] - 2026-05-29

### Added
- Graphviz DOT syntax for workflow visualization (improves LLM understanding)
- Enhanced inline def pattern clarity following Borkdude's debugging approach
- Discovery narrative that flows from observed data to hypothesis
- Explicit 3-step workflow structure emphasizing "test in nrepl before commit"

### Improved
- Better alignment with Borkdude's blog pattern: modify function with inline def → evaluate in nrepl → inspect → test fix → update source
- More concise examples while maintaining conceptual completeness (19 lines shorter)
- Clear separation of concerns: discover bug → test fixes → commit changes
- Integrant component lifecycle guidance remains clear and unchanged

### Changed
- Replaced ASCII workflow diagrams with Graphviz DOT syntax
- Reorganized example to show the "add def inside function" pattern more prominently
- Restructured fix-testing section to emphasize REPL verification before source modification

### Documentation
- Added evaluation artifacts documenting side-by-side comparison with original skill
- Added improvement summary with detailed change documentation

### Quality
- All assertions pass (5/5) for inline def pattern, discovery narrative, fix testing, code examples, and workflow structure
- Evaluated against 3 real-world debugging scenarios: calling convention bugs, missing fields, data transformations
