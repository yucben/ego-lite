# Install ego lite on Windows

Read `install.md` first; it covers upgrading an older ego lite and the onboarding step shared by every platform. This file has the Windows install steps, the commands that confirm the installation, and Windows troubleshooting.

## Run the installer

There is no install script for Windows. Have the user download the installer from https://lite.ego.app/ and run it, then open ego lite.

After the app opens, the user completes the first-run onboarding in the app (see `install.md`). Onboarding registers the `ego-browser` command on the PATH. The PATH entry only reaches processes started afterwards, so open a new terminal before checking for the command.

## After installing: confirm `ego-browser` is available

Once the user has finished onboarding, open a new terminal and confirm the command is ready. The command depends on the shell.

PowerShell:

```powershell
Get-Command ego-browser
```

cmd:

```bat
where ego-browser
```

Git Bash:

```bash
command -v ego-browser
```

Once the command exists, verify the runtime. In PowerShell or cmd, where heredoc does not exist:

```powershell
ego-browser nodejs -e "console.log('ego-browser ready')"
```

In Git Bash:

```bash
ego-browser nodejs <<'EOF'
console.log('ego-browser ready')
EOF
```

Printing `ego-browser ready` means the environment is ready. Finally confirm that the installed ego lite prints its usage guide:

```powershell
ego-browser skill
```

Then return to the user's original task, as `install.md` describes.

## Troubleshooting

- **Download fails with `0x80072ee2`**: follow [Download timeout](#download-timeout) below, including when the dialog also mentions a 64K code-segment limit.
- **SmartScreen blocks the installer**: have the user choose "More info" and then "Run anyway" in the SmartScreen dialog.
- **Command still unavailable after onboarding**: open a new terminal first, because an existing one keeps the PATH it started with. Otherwise have the user reopen ego lite, finish onboarding, and retry.

### Download timeout

[Issue #541](https://github.com/citrolabs/ego-lite/issues/541) reports a download-stage failure with `0x80072ee2` and the Chinese message `代码段不可大于或等于 64K` (a code-segment limit). The underlying Windows error code is 12002, a network request timeout ([WinINet errors](https://learn.microsoft.com/en-us/windows/win32/wininet/wininet-errors), [WinHTTP errors](https://learn.microsoft.com/en-us/windows/win32/winhttp/error-messages)). The code alone does not identify why the request timed out; the accompanying 64K message does not establish a file-size or CPU-architecture problem.

1. Record the installer filename/version, Windows version, failure stage, and complete error code. Download a fresh installer from https://lite.ego.app/ if the existing file's origin is uncertain.
2. If a proxy is in use, check it from **Windows PowerShell**, where the installer runs. Windows Settings → Network & internet → Proxy shows the user proxy. Inspect the separate WinHTTP configuration with this read-only command ([Microsoft reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netsh-winhttp)):

   ```powershell
   netsh winhttp show advproxy
   ```

   For a local proxy, confirm its configured port is listening. Replace `7890` with the Windows proxy's actual port:

   ```powershell
   Test-NetConnection -ComputerName 127.0.0.1 -Port 7890
   ```

   `TcpTestSucceeded: True` establishes only that the port accepts TCP connections. Windows and WSL may use different proxy ports; a successful download from WSL does not verify the Windows installer's network path. Different applications may also use different proxy settings. Inspect the configuration before considering changes; machine-wide WinHTTP changes affect other applications.

3. Check the proxy client's connection/error log around the failure time for failed installer requests. Successfully downloading the setup EXE only verifies that download; the installer may still need subsequent requests to finish installation. Where available, retry on another working network or proxy route and record whether the same error recurs.
4. If installation still fails, add the results to [issue #541](https://github.com/citrolabs/ego-lite/issues/541), including the failure time and relevant installer/proxy log excerpts. Redact credentials, tokens, and personal information before sharing. Report installation as successful only after the app opens, onboarding finishes, and the `ego-browser` command checks in "After installing" pass.

This repository contains the automation SDK and agent Skill; the browser app and Windows installer are maintained separately. Building the SDK here does not replace or repair the installer. Installer download handling and error-message fixes require the app maintainers; these checks help gather evidence for that work.
