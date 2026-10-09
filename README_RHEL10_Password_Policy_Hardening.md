# RHEL 10 Password Policy Hardening

## Project Overview

This project demonstrates how to configure and validate Linux password policy on Red Hat Enterprise Linux 10 (RHEL 10). It is a hands-on lab created for RHCSA (EX200) preparation and Linux system administration practice.

The project covers password aging, password quality, authentication configuration inspection, verification, troubleshooting, and rollback.

> **Lab only:** Test these changes on a virtual machine or other non-production system. Review your organization's policy before applying password rules to production systems.

## Objectives

- Inspect existing password-aging defaults.
- Configure password-aging defaults for new accounts.
- Configure password quality requirements.
- Create a dedicated test user and apply individual password-aging settings.
- Verify password-quality enforcement and PAM configuration.
- Record test results and document rollback steps.

## Lab Environment

| Item | Value |
|---|---|
| Operating system | RHEL 10 |
| Environment | Virtual machine / lab |
| Privileges | Root or sudo |
| Test account | `tux` |
| Purpose | RHCSA practice and Linux security hardening |

Update this table to reflect your actual environment.

## Policy Requirements

| Setting | Lab value | Purpose |
|---|---:|---|
| Maximum password age | 90 days | Password lifetime |
| Minimum days between changes | 1 day | Prevent immediate repeated changes |
| Expiry warning | 7 days | Warn users before expiry |
| Minimum password length | 10 characters | Minimum length |
| Uppercase characters | At least 1 | Character-class requirement |
| Lowercase characters | At least 1 | Character-class requirement |
| Digits | At least 1 | Character-class requirement |
| Special characters | At least 1 | Character-class requirement |
| Difference from previous password | 3 | Configured using `difok`, subject to module behavior |

These are the values selected for this lab, not a claim that they are the default RHEL policy.

## Implementation

### 1. Inspect the system

```bash
cat /etc/redhat-release
grep -E '^(PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE)' /etc/login.defs
authselect current
authselect check
```

Inspect the password-quality configuration and relevant PAM entry:

```bash
cat /etc/security/pwquality.conf
grep -n 'pam_pwquality.so' /etc/pam.d/system-auth
```

The `grep` command is for inspection only. Do not directly edit generated PAM files.

### 2. Configure password-aging defaults

Edit `/etc/login.defs` and set these values, replacing existing entries if present:

```text
PASS_MAX_DAYS   90
PASS_MIN_DAYS   1
PASS_WARN_AGE   7
```

Verify:

```bash
grep -E '^(PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE)' /etc/login.defs
```

These defaults primarily apply to newly created users. Existing accounts must be configured individually.

### 3. Configure password quality

Edit `/etc/security/pwquality.conf`. Set or update the following entries:

```ini
minlen = 10
difok = 3
ucredit = -1
lcredit = -1
dcredit = -1
ocredit = -1
```

Verify the active entries:

```bash
grep -E '^[[:space:]]*(minlen|difok|ucredit|lcredit|dcredit|ocredit)[[:space:]]*=' \
  /etc/security/pwquality.conf
```

These settings are effective only if the relevant authentication stack invokes the password-quality module. Verify the actual behavior in the lab.

### 4. Create a test user

```bash
useradd -m -d /home/tux -s /bin/bash tux
passwd tux
```

Set the password interactively; do not place real passwords in scripts or documentation.

### 5. Configure password aging for the test user

```bash
chage -M 90 -m 1 -W 7 tux
chage -l tux
```

### 6. Test password quality

Use the test account in a controlled lab and run:

```bash
passwd
```

Test passwords that violate the configured rules, then test a compliant password. Record the actual results in `docs/testing-results.md`.

Do not include the passwords you tried in the repository.

### 7. Verify the configuration

```bash
id tux
chage -l tux
authselect current
authselect check
```

Review the password-quality settings and confirm that the PAM stack invokes `pam_pwquality.so`.

## Test Results

Record what you actually observed. Do not mark a test as passed unless you ran it.

| Test | Expected result | Actual result |
|---|---|---|
| Password aging | Maximum 90 days, minimum 1 day, warning 7 days | TODO |
| Short password | Rejected if quality module is active | TODO |
| Missing required character class | Rejected if quality module is active | TODO |
| Compliant password | Accepted if it meets all active rules | TODO |
| Authentication profile | Profile and integrity check reviewed | TODO |

## Security Notes

- `/etc/login.defs` defines defaults for password aging; it does not enforce password complexity by itself.
- `chage` configures aging for an individual account.
- `/etc/security/pwquality.conf` configures password-quality checks.
- `pam_pwquality.so` performs quality checks when it is invoked by PAM.
- `authselect` manages supported authentication profiles on RHEL.
- Avoid directly editing generated files such as `/etc/pam.d/system-auth`.
- Password aging, password quality, and failed-login controls are separate mechanisms.

## Rollback and Cleanup

Before making changes, back up the files you plan to edit and record the original authentication profile.

For a disposable lab account, remove it after exiting its session if you no longer need it:

```bash
userdel -r tux
```

This deletes the user's home directory and data. Do not run it if you need to retain those files.

Restore configuration files from your verified backup if you need to reverse the lab. If the authentication profile was changed, restore the original supported profile and verify it using `authselect check`. Avoid manually reconstructing generated PAM files.

## Screenshots

Add screenshots to the `screenshots/` directory. Suggested evidence:

1. RHEL version and environment.
2. Password-aging defaults in `/etc/login.defs`.
3. Password-quality settings in `pwquality.conf`.
4. `chage -l tux` output.
5. Password rejection and successful validation tests.
6. `authselect current` and `authselect check` output.

Redact usernames or other information if needed. Never publish passwords, `/etc/shadow`, password hashes, access tokens, or private keys.

## What I Learned

- How Linux password aging differs from password quality.
- How to configure per-user aging with `chage`.
- How PAM and `authselect` relate to password enforcement on RHEL.
- How to verify security settings and record reproducible test results.
- Why rollback planning is important when changing authentication settings.

## Disclaimer

This repository is for educational and lab use. Validate settings against current Red Hat documentation and organizational security requirements before using them on production systems.
