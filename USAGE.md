# PSSOForge — Usage Guide

---

## PowerShell Script (`pssoforge.ps1`)

### Requirements

- **PowerShell 7+** (cross-platform — macOS and Windows 11)
- **Microsoft.Graph** module (only needed for Intune push with `-TenantId`)

### Quick Start

```powershell
# Run the interactive wizard
./scripts/pssoforge.ps1

# Specify a custom output directory
./scripts/pssoforge.ps1 -OutputPath ~/Desktop

# Load from a JSON config file (skip all questions)
./scripts/pssoforge.ps1 -InputFile config.json

# Generate and push directly to Intune
./scripts/pssoforge.ps1 -TenantId "00000000-0000-0000-0000-000000000000"

# Combine: load config + push to Intune
./scripts/pssoforge.ps1 -InputFile config.json -TenantId "00000000-0000-0000-0000-000000000000"
```

### Interactive Wizard Questions

| #  | Question | Type | Notes |
|----|----------|------|-------|
| 1  | Tenant display name | Text | Used as `AccountDisplayName` in the profile |
| 2  | Authentication method | Choice | Secure Enclave (recommended) or Password Sync |
| 3  | Run during Setup Assistant | Yes/No | Enable PSSO registration at first login |
| 4  | SAMAccountName used by LAPS | Yes/No | If yes, first-user creation is disabled |
| 5  | Main user: admin or standard | Choice | Standard is recommended |
| 6  | Multi-user Mac | Yes/No | Allow new user creation at login |
| 7  | New user: admin or standard | Choice | Only asked if Q6 = Yes |
| 8  | Managed admin account(s) to exclude | Yes/No | Exclude accounts from PSSO |
| 9  | Managed admin account name(s) | Text (multiple) | Only asked if Q8 = Yes |

### JSON Configuration File Format

Use with `-InputFile` to skip the interactive wizard:

```json
{
  "schemaVersion": 1,
  "accountDisplayName": "Contoso",
  "authenticationMethod": "UserSecureEnclaveKey",
  "enableRegistrationDuringSetup": true,
  "createFirstUserDuringSetup": false,
  "userAuthorizationMode": "Standard",
  "enableCreateUserAtLogin": false,
  "newUserAuthorizationMode": null,
  "nonPlatformSSOAccounts": ["admin", "localadmin"]
}
```

#### Field Reference

| Field | Type | Required | Values |
|-------|------|----------|--------|
| `schemaVersion` | integer | No | Default: `1` |
| `accountDisplayName` | string | Yes | Tenant display name |
| `authenticationMethod` | string | Yes | `UserSecureEnclaveKey` or `Password` |
| `enableRegistrationDuringSetup` | boolean | Yes | |
| `createFirstUserDuringSetup` | boolean | Yes | Set `false` if LAPS manages the account |
| `userAuthorizationMode` | string | Yes | `Admin` or `Standard` |
| `enableCreateUserAtLogin` | boolean | Yes | Multi-user Mac support |
| `newUserAuthorizationMode` | string | No | `Admin` or `Standard` (only if multi-user enabled) |
| `nonPlatformSSOAccounts` | string[] | No | Account names to exclude from PSSO |

### Parameters

| Parameter | Description |
|-----------|-------------|
| `-InputFile` | Path to a JSON configuration file (skips interactive wizard) |
| `-TenantId` | Azure AD tenant ID — pushes profiles directly to Intune via Microsoft Graph |
| `-OutputPath` | Custom output directory (default: current directory) |
| `-ProfileName` | Custom policy display name (default: `macOS \| PSSO <TenantName> (<SE or PSync>)`) |

### Intune Push (Microsoft Graph)

When using `-TenantId`, the script:

1. Checks for the `Microsoft.Graph` module (installs if missing)
2. Connects to Microsoft Graph with `DeviceManagementConfiguration.ReadWrite.All` scope
3. Uploads each generated profile as a **Settings Catalog configuration policy** via `POST /deviceManagement/configurationPolicies`
4. Displays the created policy ID and name for confirmation

**Install the Graph module:**

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

### Output

Each run generates one `.json` file:

```
macOS _ PSSO Contoso (SE)_2025-07-25T14_30_00.000Z.json
```

The files are **Microsoft Graph Settings Catalog JSON** (`configurationPolicies` format), ready to be imported into Microsoft Intune via the Settings Catalog policy import feature.

### Reference Documentation

- [Microsoft — Platform SSO for macOS](https://learn.microsoft.com/en-us/mem/intune/configuration/platform-sso-macos)
- [Apple — Extensible SSO MDM Schema](https://github.com/apple/device-management/blob/release/mdm/profiles/com.apple.extensiblesso.yaml)

---

## macOS Application (PSSO Forge)

> 🚧 **Coming soon** — The native macOS SwiftUI application is under development.

### Planned Features

- macOS 26+ (Tahoe) with Liquid Glass UI
- Step-by-step guided wizard matching the script's question flow
- Live preview of the generated configuration
- Export Settings Catalog JSON via save dialog
- About window with version info
- Settings window with Light / Dark / Auto appearance mode
- Links to Microsoft and Apple documentation

### Application Details

| Property | Value |
|----------|-------|
| Bundle ID | `net.e-nahum.PSSOForge` |
| Minimum OS | macOS 26 (Tahoe) |
| Framework | SwiftUI |
| UI Style | Liquid Glass |
