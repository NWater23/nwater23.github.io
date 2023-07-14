---
title: VSCode Keychain Issues
date: 2023-07-14 13:08:46
updated: 2023-07-14 13:08:46
tags:
- Linux
  - 作为环境
- 开发环境
  - IDE
    - VS Code
categories:
- 教程
  - Linux
    - 实用工具
- 笔记
  - VS Code
---

## 发现问题

更新完VSCode，突然提示出错。询问`请打开疑难解答指南以解决此问题，也可以使用不用 OS keyring 的较弱加密。`

<details>
  <summary>有关此提示的i18n信息</summary>
  

  ```json https://github.com/microsoft/vscode-loc/blob/616502b1f429c9ac0dc81004d8be2db96869f01e/i18n/vscode-language-pack-zh-hans/translations/main.i18n.json#L12221
      "vs/workbench/services/secrets/electron-sandbox/secretStorageService": {
        "encryptionNotAvailableJustTroubleshootingGuide": "无法识别用于在当前桌面环境中存储加密相关数据的 OS keyring。",
        "isGnome": "你正在 GNOME 环境中运行，但 OS keyring 不可用用于加密。请确保已安装并运行 gnome-keyring 或其他 libsecret 兼容实现。",
        "isKwallet": "你正在 KDE 环境中运行，但 OS keyring 不可用于加密。请确保 kwallet 正在运行。",
        "troubleshootingButton": "打开疑难解答指南",
        "usePlainText": "使用较弱的加密",
        "usePlainTextExtraSentence": "请打开疑难解答指南以解决此问题，也可以使用不用 OS keyring 的较弱加密。"
      },
  ```
</details>

------

## 解决问题

根据官方文档[^1]的教程，试试看日志有什么报错：
```bash
code --verbose --vmodule="*/components/os_crypt/*=1" | less
```
第二行看到了问题所在：
```
[23476:0714/131505.501724:VERBOSE1:key_storage_util_linux.cc(54)] Password storage detected desktop environment: (unknown)
```

那问题很好解决了，直接照着官方文档改一下`argv.json`就行了。
```json ~/.vscode/argv.json
{
	"password-store": "gnome",
}
```

<!-- more -->


[^1]: (2023) [Settings SYnc in Visual Studio Code§Troubleshooting keychain issues](https://code.visualstudio.com/docs/editor/settings-sync#_troubleshooting-keychain-issues)
    ## Troubleshooting keychain issues

    > NOTE: This section applies to VS Code version **1.80 and higher**. In 1.80, we moved away from [keytar](https://github.com/atom/node-keytar), due to its archival, in favor of Electron's [safeStorage API](https://www.electronjs.org/docs/latest/api/safe-storage).

    > NOTE: keychain, keyring, wallet, credential store are synonymous in this document.

    Settings Sync persists authentication information on desktop using the OS keychain for encryption. Using the keychain can fail in some cases if the keychain is misconfigured or the environment isn't recognized.

    To help diagnose the problem, you can restart VS Code with the following flags to generate a verbose log:

    ```
    code --verbose --vmodule="*/components/os_crypt/*=1"
    ```

    ### Windows & macOS

    At this time, there are no known configuration issues on Windows or macOS but, if you suspect something is wrong, you can open an [issue on VS Code](https://github.com/microsoft/vscode/issues/new/choose) with the verbose logs from above. This is important for us to support additional desktop configurations.

    ### Linux

    Towards the top of the logs from the previous command, you will see something to the effect of:

    ```
    [9699:0626/093542.027629:VERBOSE1:key_storage_util_linux.cc(54)] Password storage detected desktop environment: GNOME
    [9699:0626/093542.027660:VERBOSE1:key_storage_linux.cc(122)] Selected backend for OSCrypt: GNOME_ANY
    ```

    We rely on Chromium's oscrypt module to discover and store encryption key information in the keyring. Chromium supports [a number of different desktop environments](https://source.chromium.org/chromium/chromium/src/+/main:base/nix/xdg_util.cc;l=146-169). Outlined below are some popular desktop environments and troubleshooting steps that may help if the keyring is misconfigured.

    #### GNOME or UNITY (or similar)

    If the error you're seeing is "Cannot create an item in a locked collection", chances are your keyring's `Login` keyring is locked. You should launch your OS's keyring ([Seahorse](https://wiki.gnome.org/Apps/Seahorse) is the commonly used GUI for seeing keyrings) and ensure the default keyring (usually referred to as `Login` keyring) is unlocked. This keyring needs to be unlocked when you log into your system.

    #### KDE

    > KDE 6 is not yet fully supported by Visual Studio Code. As a workaround: The latest kwallet6 is also accessible as kwallet5, so you can force it to use kwallet5 by setting the password store to `kwallet5` as explained below in [Configure the keyring to use with VS Code](#other-linux-desktop-environments).

    It's possible that your wallet (aka keyring) is closed. If you open [KWalletManager](https://apps.kde.org/kwalletmanager5), you can see if the default `kdewallet` is closed and if it is, make sure you open it.

    #### Other Linux desktop environments

    First off, if your desktop environment wasn't detected, you can [open an issue on VS Code](https://github.com/microsoft/vscode/issues/new/choose) with the verbose logs from above. This is important for us to support additional desktop configurations.

    #### (recommended) Configure the keyring to use with VS Code

    You can manually tell VS Code which keyring to use by passing the `password-store` flag. Our recommended configuration is to first install [gnome-keyring](https://wiki.gnome.org/Projects/GnomeKeyring) if you don't have it already and then launch VS Code with `code --password-store="gnome"`.

    If this solution works for you, you can persist the value of `password-store` by opening the Command Palette (`kb(workbench.action.showCommands)`) and running the **Preferences: Configure Runtime Arguments** command. This will open the `argv.json` file where you can add the setting `"password-store":"gnome"`.

    > NOTE: If you would rather not use `gnome-keyring`, you can try using a package that implements the [Secret Service API](https://www.gnu.org/software/emacs/manual/html_node/auth/Secret-Service-API.html). If you do this, the `password-store` flag can still be set to `gnome` and Electron will detect other implementations of the Secret Service API. Additionally, you could try installing `kwallet5` on your system. If you do, you will want to set the `password-store` flag to `kwallet5` to detect the installed `kwallet5`. All possible values for `password-store` can be [found in Chromium's source](https://source.chromium.org/chromium/chromium/src/+/refs/tags/108.0.5359.215:components/os_crypt/key_storage_util_linux.cc;l=35-46).

    Don't hesitate to [open an issue on VS Code](https://github.com/microsoft/vscode/issues/new/choose) with the verbose logs if you run into any issues.

    #### (not recommended) Configure basic text encryption

    We rely on Chromium's oscrypt module to discover and store encryption key information in the keyring. Chromium offers an opt-in fallback encryption strategy that uses an in-memory key based on a string that is hardcoded in the Chromium source. Because of this, this fallback strategy is, at best, obfuscation, and should only be used if you are accepting of the risk that any process on the system could, in theory, decrypt your stored secrets.

    If you accept this risk, you can set `password-store` to `basic` by opening the Command Palette (`kb(workbench.action.showCommands)`) and running the **Preferences: Configure Runtime Arguments** command. This will open the `argv.json` file where you can add the setting `"password-store":"basic"`.
[^2]: (2023) [github.com/microsoft/vscode-log/i18n/vscode-language-pack-zh-hans/translations/main.i18n.json:12221](https://github.com/microsoft/vscode-loc/blob/616502b1f429c9ac0dc81004d8be2db96869f01e/i18n/vscode-language-pack-zh-hans/translations/main.i18n.json#L12221)