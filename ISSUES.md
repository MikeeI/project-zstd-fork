# Issue and Pull Request Tracking

Read this index at the start of every agent session before repository work.
`FORMAT.md` owns research, lifecycle, drafting, implementation, and publication rules.
Each linked `issues/ISSUE-NNN.md` is the complete authoritative record for one root cause.
This file owns `Next finding ID` and projects current issue-file state.
`Next-Action` is the 2–6 word `Next-Action/Summary` projection from the issue record.
When a row disagrees with its issue file, correct the row from the issue file in the same task.
Apply the finding eligibility rules in `AGENTS.md`; tuning tips and reporting lessons do not belong in Open-Findings.

Next finding ID: ISSUE-008

## Open-Findings

| ID | Finding | State | Authorized-Work | Publication-Target | Contribution-Priority | Next-Action | External-Reference |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-001](issues/ISSUE-001.md) | CLI: skip the unused batch size prepass outside patch mode | Investigating | Not-Selected | Not-Selected | Low | Select scoped implementation authorization | Not published. |
| [ISSUE-002](issues/ISSUE-002.md) | CLI: avoid zero-distance seeks in automatic sparse decompression | Investigating | Not-Selected | Not-Selected | Low | Select scoped implementation authorization | Not published. |
| [ISSUE-004](issues/ISSUE-004.md) | Huffman: preserve compression opportunities for small alphabets | Implementing | Pull-Request-Implementation | Not-Selected | Medium | Verify scoped encoder correction | Not published. |
| [ISSUE-005](issues/ISSUE-005.md) | FSE: avoid basic encoding for two identical sequence codes | Implementing | Pull-Request-Implementation | Not-Selected | Medium | Verify scoped encoder correction | Not published. |
| [ISSUE-006](issues/ISSUE-006.md) | Compression: encode short repeated follow-up blocks as RLE | Implementing | Pull-Request-Implementation | Not-Selected | Low | Verify scoped encoder correction | Not published. |
| [ISSUE-007](issues/ISSUE-007.md) | Compression: include Raw/RLE literal headers in block-size estimates | Investigating | Not-Selected | Not-Selected | Low | Establish affected sizing decisions | Not published. |

## Archived-Findings

| ID | Finding | Authorized-Work | Publication-Target | Contribution-Priority | Archive-Reason | External-Reference |
| --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-003](issues/archive/ISSUE-003.md) | Compression level comparison (not a repository issue) | Not-Selected | Not-Selected | Low | Finding-Invalidated | Not published. |
