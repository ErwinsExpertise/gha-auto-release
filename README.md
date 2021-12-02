# gha-auto-release

## Releasing new builds

New tags will **not** be created unless "Release v`<version>` #`<type>`" is present in the commit message.

* Examples
  * Release v0.2.0 #minor
  * Release v1.2.0 #major
  * Release v1.2.1 #patch

Failing to add the release string into the message will prevent the actions from running.

### Upgrade strategy

The default upgrade strategy is set at minor upgrades. You can adjust this by adding the appropriate string to your commit message.

Available options working from v0.0.0

| String | Type | Outcome |
|--|--|--|
| #major | x.0.0 | v1.0.0 |
| #minor | 0.x.0 | v0.1.0 |
| #patch | 0.0.x | v0.0.1 |  

- Example
  - git commit -m "Release v1.0.0 #major"

### GoReleaser

For the most part GoReleaser strategies can be added to the workflow as normal. They will need to be added directly to the workflow because at this time you cannot fire off a GHA workflow from another. 


# Workflow


```
name: "Automatic Releaser"

on:
  push:

permissions:
  contents: write

jobs:
  check-commit:
    runs-on: ubuntu-latest
    outputs:
      msg_check: ${{ steps.check-msg.outputs.match }}
    steps:
      - name: Check Message
        id: check-msg
        run: |
          pattern="^Release v[0-9].[0-9].[0-9] #(minor|major|patch)$"
          if [[ "${{ github.event.head_commit.message }}" =~ ${pattern} ]]; then
              echo ::set-output name=match::true
          fi

  create-tag:
    runs-on: ubuntu-latest
    if: needs.check-commit.outputs.msg_check == 'true'
    needs: check-commit
    outputs:
      new_tag: ${{ steps.tagger.outputs.new_tag }}
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: '0'

    - name: Bump version and push tag
      id: tagger
      uses: anothrNick/github-tag-action@1.36.0
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        WITH_V: true
        DEFAULT_BUMP: "none"

  goreleaser:
    runs-on: ubuntu-latest
    needs: create-tag
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
      -
        name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.17
      -
        name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v2
        with:
          distribution: goreleaser
          version: latest
          args: release --rm-dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```