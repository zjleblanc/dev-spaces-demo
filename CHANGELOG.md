# Changelog

## 2026-11-16 — Set up SSH keys for ansible-navigator

### Added

- README steps to create an OpenShift secret from the Ansible SSH key pair.
- A Dev Spaces post-start command that copies the mounted keys into `/projects/.ssh` and sets permissions so ansible-navigator can use them.
