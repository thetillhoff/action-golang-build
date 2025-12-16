# Action: Golang build

This action builds golang executables for multiple operating systems and architectures and stores them as artifacts.

## Features

- Builds Go binaries for multiple OS/architecture combinations
- Generates SHA256 checksums for all binaries
- Uploads binaries and checksums as GitHub artifacts

## Inputs

| Input       | Description                                                                                          | Required | Default                               |
| ----------- | ---------------------------------------------------------------------------------------------------- | -------- | ------------------------------------- |
| `APP_NAME`  | Name of the application                                                                              | No       | `${{ github.event.repository.name }}` |
| `OS`        | Operating system to build for (`linux`, `windows`, `darwin`)                                         | Yes      | -                                     |
| `ARCH`      | Architecture to build for (e.g., `amd64`, `arm64`)                                                   | Yes      | -                                     |
| `BUILDARGS` | Additional arguments for `go build` (e.g., `-ldflags='-X github.com/owner/repo/cmd.version=v1.0.0'`) | No       | `''`                                  |

## Example usage

### Basic usage with matrix strategy

```yaml
jobs:
  golang-build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os:
          - linux
          - windows
          - darwin
        arch:
          - amd64
          - arm64
    steps:
      - uses: actions/checkout@v5
      - uses: thetillhoff/action-golang-build@v0.2.0
        with:
          OS: '${{ matrix.os }}'
          ARCH: '${{ matrix.arch }}'
          BUILDARGS: -ldflags="-X 'github.com/<owner>/<repo>/cmd.version=${{ github.ref_name }}'"
```

### PR checks (read-only permissions)

```yaml
on:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: thetillhoff/action-golang-build@v0.2.0
        with:
          OS: 'linux'
          ARCH: 'amd64'
```

### Tag releases

```yaml
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: [linux, windows, darwin]
        arch: [amd64, arm64]
    steps:
      - uses: actions/checkout@v5
      - uses: thetillhoff/action-golang-build@v0.2.0
        with:
          OS: '${{ matrix.os }}'
          ARCH: '${{ matrix.arch }}'
          BUILDARGS: -ldflags="-X 'github.com/<owner>/<repo>/cmd.version=${{ github.ref_name }}'"
```

## Deleting tags on build failure

If you want to delete a tag when the build fails (e.g., for tag-based releases), you can add a cleanup step to your workflow. This step should run after the build step and only on tag pushes:

```yaml
on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write # Required for tag deletion
    strategy:
      matrix:
        os: [linux, windows, darwin]
        arch: [amd64, arm64]
    steps:
      - uses: actions/checkout@v5
      - uses: thetillhoff/action-golang-build@v0.2.0
        with:
          OS: '${{ matrix.os }}'
          ARCH: '${{ matrix.arch }}'

      - name: Delete tag on failure
        if: failure() && github.ref_type == 'tag' && github.event_name == 'push'
        shell: bash
        run: |
          git config --global user.name 'GithubActions'
          git config --global user.email 'githubactions@users.noreply.github.com'
          git push --delete origin "${{ github.ref_name }}"
```

**Note:** This step requires `contents: write` permissions and will only run on tag pushes when the build fails. The action itself only needs `contents: read` permissions for building.
