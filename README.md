# GitHub Workflows

This repository houses reusable workflows used by GitHub Actions.

The main objective of such a repository is centralizing all default workflows used by applications in the IT
environment, allowing for better management of build and deploy pipelines, or dependency publishing.

## How it works

**NOTICE!** The repository that houses the reusable workflows must allow them to be used by other repositories
within the organization (configurable in repository settings, on the _Actions > General_ section).

For any new reusable workflow, it must be created in the ___.github/workflows___ directory.

The following sections explains how reusable workflows are used.

### Creating a reusable workflow

The main distinguishable feature of a reusable workflow is the **on:** tag, where all reusable workflows have the
_workflow_call_ parameter, as shown in the following example:

```yaml
name: Example workflow

on: [ workflow_call ]

jobs:
  job-name:
```

Then, in the **jobs:** tag, jobs and steps are declared as any other runnable job.

### Adding inputs

Reusable workflows can have inputs that can be used within its execution, as shown below:

```yaml
name: Example workflow

on:
  workflow_call:
    inputs:
      some-input:
        default: ...
        description: ...
        required: ...
        type: ...

jobs:
  job-name:
```

### Reusing a workflow

A job can call a reusable workflow as follows:

```yaml
name: Caller example workflow

# Definition for which events this workflow will run
on: [ push, pull_request ]

jobs:
  job-name:
    uses: <REPOSITORY_OWNER>/<REPOSITORY_NAME>/workflow-name.yml@<BRANCH>
```

Inputs and secret variables can also be passed to the reusable workflow (if defined in the latter):

```yaml
name: Caller example workflow

# Definition for which events this workflow will run
on: [ push, pull_request ]

jobs:
  job-name:
    uses: <REPOSITORY_OWNER>/<REPOSITORY_NAME>/workflow-name.yml@<BRANCH>
    with:
      some-input: ...
    # Secret variables can be sent as needed (with variables), or the entirety of them (with inherit)
    secrets: inherit
```

### Known limitations

* Workflows can be connected up to four levels;
* Reusable workflows doesn't have access to the environment variables from the caller workflow, requiring the use of
  inputs as a workaround;
* To reuse variables in multiple workflows, set them at the organization, repository, or environment levels, and
  reference them using the appropriate context.

---

### Example

The detailed example below shows how reusable workflows are called, with all the features explained in previous steps.

___reusable-workflow.yml___

```yaml
name: Reusable workflow

on:
  workflow_call:
    inputs:
      my-input:
        default: Hello world!
        required: false
        type: string
      working-directory:
        required: true
        type: string

# Declares environment variables for entire job
# Secrets and variables saved in repository settings
env:
  SECRET-VARIABLE: ${{ secrets.SECRET-INPUT }}
  ENV-VARIABLE: ${{ vars.ENV-INPUT }}

jobs:
  my-job:
    uses: ubuntu-latest
    steps:
      # Default GitHub Action to pull running branch from repository
      - uses: actions/checkout@v4
      - name: List current working directory
        # Multiline ordered execution
        run: |
          echo "${{ inputs.working-directory }}"
          ls -al ./
      - name: Show input parameter
        run: echo "${{ inputs.my-input }}"
      - name: Show environment variable 1
        run: echo "${{ env.SECRET-VARIABLE }}"
      - name: Show environment variable 2
        run: echo "${{ env.ENV-VARIABLE }}"

  environment-job:
    # Determines which jobs must be successful before running
    needs: [ my-job ]
    uses: ubuntu-latest
    # Environment defined in repository settings
    environment: development
    concurrency: development
    steps:
      - name: Show input parameter
        run: echo "${{ inputs.my-input }}"
      # Secrets defined in environment
      - name: Show environment secret parameter
        run: echo "${{ secrets.ENVIRONMENT-SECRET }}"
      # Variables defined in environment
      - name: Show var context environment variable
        run: echo "${{ vars.ENVIRONMENT-VARIABLE }}"
```

___main-workflow.yml___

```yaml
name: Main workflow (calling reusable workflow)

on: [ push, pull_request ]

jobs:
  # Workaround for jobs that send environment variables to reusable workflow
  environment-variables:
    runs-on: ubuntu-latest
    outputs:
      working-directory: ${{ steps.environment-variables.outputs.working-directory }}
    steps:
      id: environment-variables
      run: echo "working-directory=${GITHUB_WORKSPACE}" >> $GITHUB_OUTPUT

  main-job:
    needs: [ environment-variables ]
    uses: <REPOSITORY_OWNER>/<REPOSITORY_NAME>/reusable-workflow.yml@main
    # Inputs for reusable workflow
    with:
      my-input: My new input!
      # Reads output from previous job (environment-variables)
      working-directory: ${{ needs.environment-variables.outputs.working-directory }}
    # Sends all secrets from job caller to the reusable workflow
    secrets: inherit
```

---

### Related links

[About workflows (GitHub Docs)](https://docs.github.com/en/actions/using-workflows/about-workflows)  
[Workflow syntax (GitHub Docs)](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)  
[Environment variables (GitHub Docs)](https://docs.github.com/en/actions/learn-github-actions/environment-variables)  
[Reusable workflows (GitHub Docs)](https://docs.github.com/en/actions/using-workflows/reusing-workflows)  
[Contexts (GitHub Docs)](https://docs.github.com/en/actions/learn-github-actions/contexts)  
[Delete Package Versions (Actions)](https://github.com/actions/delete-package-versions)
