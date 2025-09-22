# GitHub

## Actions

You can split a chunk of logic into a reusable separate **composite action**.

Action should be located in an `action-name/action.yml` folder.

This is the layout of a composite action with some examples:

```yml
name: action-name
inputs:
  folder:
    description: Working directory of .NET solution
    required: true
permissions:
  contents: read
runs:
  using: composite
  steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Test solution
      shell: bash
      run: dotnet test
      working-directory: ${{ inputs.folder }}
```

For `uses` steps, you do not specify `shell`. For `run` steps you specify `shell`.
