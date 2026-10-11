Read AGENTS-shape.md first — the rules of this repository's shape.

This is the assembly root of **openRepoKit** (`openrepokit`): the repository an
engineer clones. It holds `project.yaml`, the pins and the gate, and it mounts
the two legs.

| role | repository | path |
|---|---|---|
| assembly | `opensoft/openRepoKit` | `.` |
| spec | `opensoft/openRepoKit-spec` | `spec/` |
| code | `opensoft/openRepoKit-code` | `code/` |

Everything below this line is openRepoKit's own. The shape wrote the block above
once and does not pin this file, so how this project is built, tested, reviewed
and released belongs here and nothing upstream will overwrite it.

Run `make validate`, `python3 code/openRepoKit verify`, and
`python3 -m unittest discover -s code/tests -v` from this root.
Contracts belong in `spec/contracts`; implementation and tests belong in code.
Never edit component mounts: land component changes upstream and deliberately
advance the code gitlink and spec contract. Preserve Tools' nested Shape pin.
The integration gate must fail if recursive checkouts are unavailable.
Use openRepoShape's `bump-leg.py` to advance leg pins, never hand-edit digests.
Installation tests use temporary prefixes and isolated Claude configuration.
