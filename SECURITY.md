# Security Policy

**Archive Preview** is a Windhawk mod for the Windows File Explorer Preview Pane (`Alt + P`). It lets users browse, open, copy, drag, and extract archive and compressed files, including ZIP, RAR, 7z, and TAR formats.

Archives can contain harmful files or unexpected data. The mod also works with Windows Explorer and Preview Host processes, so security reports are important.

## Supported versions

| Version | Security support |
| --- | --- |
| 1.0.x (after public release) | Reports are accepted. Confirmed issues will be reviewed and fixed on a best-effort basis. |
| Older releases | Security fixes are not guaranteed. Update when a fixed version is available. |
| Development or test builds | Reports are welcome, but these builds are not considered production-validated. |

This policy reflects the **Archive Preview v1.0.0 source (R140 B76)**. Static checks and isolated tests do not replace a full Windows/Windhawk integration test or an independent security review. This policy does not promise that the software has no vulnerabilities.

## Report a vulnerability

**Do not share exploit steps, private files, or security-sensitive details in public issues, discussions, or pull requests.**

1. Open the repository's **Security** tab and choose **Report a vulnerability**, if private vulnerability reporting is enabled.
2. If that option is not available, use a private contact method listed on the [maintainer's GitHub profile](https://github.com/luizbizzio).
3. If there is no private contact method, open a public issue asking for a private reporting channel. Do not include the vulnerability details in that issue.

When possible, include:

- The mod version, Windows build, Windhawk version, and affected archive format.
- The action involved, such as preview, browsing, opening, drag and drop, `Ctrl + C`, extraction, cache cleanup, installation, or removal.
- Steps to reproduce the issue, the expected result, what happened, and the possible security impact.
- Any settings or Windows archive providers needed to reproduce it.
- Logs with personal data removed. Share proof-of-concept archives only through a private channel when needed.

For normal bugs, crashes, and feature requests without a security impact, use [GitHub Issues](https://github.com/luizbizzio/archive-preview/issues) instead.

## Security areas

Reports are especially useful for:

- **Archive paths and extraction:** path traversal, unsafe file names, alternate data streams, symbolic links, junctions, or writes outside the selected destination.
- **Unexpected execution:** opening or running a program without a clear user action, or bypassing attachment checks and the EXE warning.
- **Windows processes and IPC:** unauthorized requests between `prevhost.exe` and the broker in `explorer.exe`, including named-pipe access and request checks.
- **Temporary files and cache:** data exposure, stale or modified files, unsafe cleanup, lease errors, and deletion of files still in use.
- **Race conditions:** a source archive, selection, path, cache entry, or destination changing while an operation is running.
- **Resource limits:** archives that bypass size, depth, memory, disk, or process limits.
- **Installation and removal:** unsafe changes to Windows preview associations or unexpected Explorer restarts.
- **External providers:** security issues related to Windows Shell archive providers or the Windows `System32\tar.exe` fallback.

## Security design and limitations

The following describes the v1.0.0 implementation. These controls reduce risk but **do not guarantee that all attacks are prevented**.

- The preview handler runs through Windows Preview Host. A separate broker in `explorer.exe` performs file operations. Requests use bounded IPC and process/client checks.
- The mod checks archive paths, extracted entry names, file types, and temporary directories. It includes checks for reparse points and uses protected staging paths in several operations.
- Extraction uses Windows Shell archive providers or a bounded `tar.exe` fallback. Archive support depends on the Windows version and installed providers.
- Default limits include **2 GiB** for automatic pre-extraction per selection, **30 GiB** for drag/copy/open operations, **60 GiB** for manual extraction, and **160 GiB** for the combined temporary workspace. These are configurable within restricted ranges. A limit does not guarantee that disk use will be exactly equal to the reported file size.
- **Actions / Skip EXE warning** is **off by default**. The mod normally asks for confirmation before opening an EXE from an archive. Windows security checks may also apply, but UAC, antivirus, and SmartScreen do not guarantee safety.
- **Security / Allow preview for downloaded archives** is **off by default**. Enabling it may relax a Windows preview security check for supported archives on local fixed drives. It does not remove Mark of the Web or change system-wide policies, and it may not work on every Windows build. Leave it off for archives you do not trust.
- Active transfers, clipboard files, and certain opened files can keep temporary stages protected. The mod uses cache pins and time-based lease records. An expired lease does not prove that another application has finished using the files.
- Cancellation is cooperative. A blocked Shell/COM provider can still require restarting File Explorer.
- There is **no password-entry dialog** for encrypted archives. Some password-protected archives cannot be opened or extracted.

**Previewing an archive does not make its contents safe.** Only open or run extracted programs from sources you trust.

## Temporary files, logs, and privacy

The v1.0.0 source still uses **`CompressedFilePreview`** as its internal data-folder name. This is intentional, even though the mod is now named *Archive Preview*.

| Location | Purpose |
| --- | --- |
| `%LOCALAPPDATA%\CompressedFilePreview\PrefetchCache` | Automatic pre-extraction |
| `%LOCALAPPDATA%\CompressedFilePreview\TransferCache` | Temporary data for drag, copy, and opened files |
| `%LOCALAPPDATA%\CompressedFilePreview\ExtractionWork` | Manual extraction work |
| `%LOCALAPPDATA%\CompressedFilePreview\StageLeases` | Records used to protect staged files |
| `%LOCALAPPDATA%\CompressedFilePreview\transfer-broker.log` | Broker diagnostics |
| `%USERPROFILE%\AppData\LocalLow\CompressedFilePreview\drag-source.log` | Preview Host drag diagnostics |
| `%TEMP%\CompressedFilePreview` | Temporary `.tbz` aliases |

The `LocalLow` location is resolved through Windows known folders, so its exact path may differ between systems.

Logs may contain local paths, archive entry names, and operation details. Review them and remove private information before sharing. Temporary stages are **not secure long-term storage**. Do not manually delete active cache files while another application may still need them.

## Disclosure and response

Reports are reviewed on a **best-effort basis**. There is no guaranteed response time or fix deadline. When possible, please allow time for a fix before publishing technical details. Reporter credit can be provided with permission.

## Safer use

- Use an official Windhawk release or the project's source repository.
- Keep Windows, Windhawk, and archive providers updated.
- Keep **Skip EXE warning** disabled unless you understand the risk.
- Keep **Allow preview for downloaded archives** disabled unless you accept the added risk.
- Avoid opening executables from archives you do not trust.
- Test security issues only on systems and data you own or are allowed to test.

## License and third-party components

The mod uses the **Apache License 2.0** and includes attribution for MIT-licensed interface ideas adapted from another Windhawk mod. Windows Shell providers and `tar.exe` are external components. These licenses and dependencies do not provide a security warranty.
