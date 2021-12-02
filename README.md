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