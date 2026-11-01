# Security Policy

**Compressed File Preview** is a Windhawk mod that previews and browses compressed archives in the Windows File Explorer Preview Pane (`Alt + P`). It can also open, copy, drag, and extract archived files. Because archives can contain untrusted data and the mod integrates with Windows processes, security reports are taken seriously.

## Supported versions

| Version | Security support |
| --- | --- |
| Latest published stable release | Security reports are accepted, and confirmed issues are addressed on a best-effort basis. |
| Older releases | Not guaranteed to receive security fixes. Update to a fixed release when available. |
| Development or stability-test builds | Reports are welcome, but these builds are not considered production-validated. |

This policy was updated using the **v0.9.3, R91 B23** source revision. That revision is marked **Stability Test**, and its source notes that the patch had **not been compiled or tested on Windows** at the time it was prepared. This policy is not a statement that the build has passed a security audit.

## Reporting a vulnerability

**Do not post security-sensitive details or exploit files in a public issue, discussion, or pull request.**

1. Open the repository's **Security** tab and use **Report a vulnerability** if private vulnerability reporting is enabled.
2. Otherwise, use a private contact method listed on the maintainer's GitHub profile.
3. If no private channel is available, open a public issue asking for a private reporting channel, **without including vulnerability details**.

Please include the following when possible:

- Mod version and source revision, Windows build, Windhawk version, and relevant archive format.
- The action involved: preview, browsing, opening, pre-extraction, drag and drop, `Ctrl+C`, manual extraction, installation, or removal.
- Reproduction steps, expected and actual results, and possible security impact.
- Whether the issue needs a crafted archive, specific settings, a third-party Shell provider, or user interaction.
- A minimal proof of concept or test archive shared **privately**, only if needed and safe to provide.
- Sanitized logs or error details. Remove personal paths, usernames, filenames containing sensitive information, and private document contents.

If user data may be at risk or exploitation is ongoing, make that clear in the report. Ordinary crashes and compatibility problems without a security impact should be reported through GitHub Issues instead.

## Security-relevant areas

Reports are especially useful for issues involving:

- **Archive and destination paths:** path traversal, absolute paths, unsafe names, alternate data streams, symbolic links, junctions, reparse points, or writes outside an intended destination.
- **Unexpected execution:** opening an executable without a deliberate action, bypassing the mod's EXE confirmation, executing blocked file types, or bypassing Windows attachment protections.
- **Process and IPC boundaries:** unauthorized requests crossing between `prevhost.exe` and the transfer Broker in `explorer.exe`, named-pipe access control, client checks, or request authorization.
- **Temporary-file safety:** exposure or modification of staged content, reuse of stale files, unsafe cleanup, or corrupted cache and lease metadata.
- **Race conditions:** changing archives, staged files, selections, or destinations during validation, extraction, transfer, or cleanup.
- **Resource exhaustion:** crafted archives that bypass configured size, depth, workspace, or process limits in a way that causes meaningful security impact.
- **Registration and lifecycle:** unsafe changes to preview-handler associations, unexpected actions during update, uninstall, or Explorer restart.
- **External components:** exploitable behavior in an invoked Windows Shell archive provider or the Windows `System32\tar.exe` fallback.

## Security design and limitations

The following describes controls visible in the **R91 B23 source**. These are implementation details, **not guarantees** that every attack is prevented.

- A native preview handler integrates with `prevhost.exe`; a separate Broker inside `explorer.exe` handles privileged file operations.
- Broker requests use a local named pipe with access restrictions, client process/session checks, preview-window authorization, and bounded request handling.
- Archive and relative entry paths are validated, and code paths reject unsafe names and reparse points in several file operations.
- The mod uses Windows Shell archive providers and, when needed, the system `tar.exe`. The fallback extracts selected entry data through bounded output processing rather than directing `tar.exe` to extract an entire archive to an arbitrary path.
- Prefetch, transfer, and extraction have separate configurable limits. The default global temporary workspace budget is **120 GiB**. Limits reduce some risks but do not guarantee that an archive is safe or that disk use will equal the reported logical size.
- The mod applies Windows Attachment Manager policy before opening a staged file. Certain dangerous file types are blocked from the mod's Open action. Opening an `.exe` requires an additional confirmation by default; the **Skip EXE confirmation popup** setting disables only that extra confirmation.
- File transfers and opened files may remain in staging folders for a limited time so other applications can continue using them. A lease expiration is **not proof** that a receiving application has finished using a file.
- Cancellation is cooperative. A third-party Shell/COM provider may hang inside `explorer.exe` despite the mod's timeouts and cancellation handling.

**Opening a compressed archive is not a safe-content guarantee.** Keep Windows and its archive providers updated, and do not run extracted programs from untrusted sources. A file can be dangerous even if extraction succeeds.

## Temporary files, logs, and privacy

Starting with the **R91 B23 source**, the relevant paths are:

| Location | Purpose |
| --- | --- |
| `%LOCALAPPDATA%\CompressedFilePreview\transfer-broker.log` | Broker diagnostics |
| `%LOCALAPPDATA%\CompressedFilePreview\PrefetchCache` | Speculative pre-extraction |
| `%LOCALAPPDATA%\CompressedFilePreview\TransferCache` | Temporary drag, copy, and open-file data |
| `%LOCALAPPDATA%\CompressedFilePreview\ExtractionWork` | Temporary manual-extraction work |
| `%LOCALAPPDATA%\CompressedFilePreview\StageLeases` | Persistent staging retention records |
| `%LOCALAPPDATALOW%\CompressedFilePreview\drag-source.log` | Low-integrity preview drag diagnostics (normally under `AppData\LocalLow`) |
| `%TEMP%\CompressedFilePreview` | Temporary `.tbz` browsing aliases |

`%LOCALAPPDATALOW%` above describes the location resolved through Windows **FOLDERID_LocalAppDataLow**; it is not guaranteed to be a defined environment variable on every system.

**Compatibility note:** R91 B23 does **not** automatically migrate or delete data from older `NativeArchivePreview` and `ArchivePreviewPane` directories. An older clipboard or Explorer operation might still reference those files. Only review or remove old temporary data when it is no longer in use.

Diagnostic logs may contain local file paths, archive entry names, and operation details. Review and redact them before sharing. Do not post private archives, unsanitized dumps, or confidential staged files publicly. Staging folders are temporary work areas, **not secure long-term storage**. Do not manually delete active stages while another application may be using them.

## Disclosure and response

Reports will be reviewed on a **best-effort basis**. There is no guaranteed response time, security-fix deadline, or service-level agreement. Where practical, please coordinate public disclosure so a fix can be prepared and users can update. Reporter credit can be provided with permission.

## Safer use and testing

- Use the official Windhawk catalog or the project's official source repository, and install the latest stable release.
- Keep Windows and Windhawk updated, especially when using third-party archive providers.
- Keep **Skip EXE confirmation popup** disabled unless you understand the risk.
- Avoid opening or running files from archives you do not trust.
- Test suspected vulnerabilities only on devices and data you own or are authorized to test.
- Do not post weaponized archives, private user data, or sensitive proof-of-concept details in public issues.

## License and third-party components

The reviewed source declares the mod's code under **Apache License 2.0** and retains attribution for MIT-licensed theme-related code adapted from another Windhawk mod. Windows Shell providers and the system `tar.exe` are external components. These licenses and dependencies do not imply a security warranty.
