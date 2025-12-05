# Action: Golang build

This action builds golang executables and stores them as artifacts.

## Example usage

```yaml
# ...

jobs:
  # ...

  golang-build:
    needs: release-prerequisites
    runs-on: ubuntu-latest
    permissions:
      contents: write
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
      - uses: thetillhoff/action-golang-build@v0.2.0
        with:
          OS: "${{ matrix.os }}"
          ARCH: "${{ matrix.arch }}"
          BUILDARGS: -ldflags="-X 'github.com/<owner>/<repo>/cmd.version=${{ github.ref_name }}'"

  # ...
```
