# Bash

Start scripts with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

or

```bash
#!/usr/bin/env bash
set -o errexit  # abort on nonzero exitstatus
set -o nounset  # abort on unbound variable
set -o pipefail # don't hide errors within pipes
```

Check for syntax errors:

```bash
bash -n myscript.sh
```

Linter: `shellcheck` (vscode-shellcheck)
