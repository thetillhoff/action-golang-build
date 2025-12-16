# CHANGELOG

## v1.0.0

- **Breaking:** Removed checkout step - workflows must now checkout code before using this action
- **Breaking:** Removed tag deletion functionality - see README for how to add it as a separate step in workflows
- Simplified action to focus on building only
- Works with read-only permissions for PR checks
- Removed unused inputs (REF, VERSION, DELETE_TAG_ON_FAILURE)

## v0.3.0

- Add CGO_ENABLED=0 to the build command for static linking by default

## v0.2.0

- Update dependencies

## v0.1.0

Initial release
