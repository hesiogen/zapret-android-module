# Compilation Guide

To compile `zapret-pocket` yourself, you have two options: using GitHub Actions (easiest) or building locally.

## Option 1: GitHub Actions (Recommended)

1.  **Fork** this repository to your GitHub account.
2.  Go to the **Actions** tab in your forked repository.
3.  Enable Workflows if prompted.
4.  Create a new **Release** or **Tag** (e.g., `v22.0`).
    - The workflow is triggered by tags matching `[0-9]+*`.
5.  Wait for the `Build Zapret Pocket` workflow to complete.
6.  The compiled `zapret-pocket.zip` will be available in the **Releases** section of your repo.

## Option 2: Local Compilation

Building locally requires a Linux environment with the Android NDK and standard build tools.

### Prerequisites
*   Android NDK (r26 or newer recommended)
*   `make`, `curl`, `tar`, `git`, `7zip` (p7zip-full)
*   Go (for `dnscrypt-proxy`)

### Steps

1.  **Clone the repositories:**
    ```bash
    git clone https://github.com/bol-van/zapret.git upstream_zapret
    # Clone this repo as well if you haven't
    ```

2.  **Build `nfqws` (Zapret) for Android:**
    You need to set up the Android NDK environment variables (`ANDROID_NDK_HOME`, etc.) and cross-compile `libnetfilter_queue`, `libmnl`, `libnfnetlink`.

    Refer to `.github/workflows/build_module.yml` for the exact build commands and patches required.

    Basically:
    - Compile dependencies (`libmnl`, `libnfnetlink`, `libnetfilter_queue`) for each architecture (arm, arm64, x86, x86_64).
    - Run `make android` in `upstream_zapret`.

3.  **Build `dnscrypt-proxy`:**
    - Use Go to cross-compile `dnscrypt-proxy` for android/arm, android/arm64, etc.

4.  **Build `curl`:**
    - Cross-compile `curl` using the NDK toolchain and OpenSSL.

5.  **Assemble the Module:**
    - Place the compiled binaries into the `module/` folder structure:
        - `module/zapret/nfqws` (rename the binary for the target arch to just `nfqws` before zipping, or let the `customize.sh` script handle it if you include all archs named correctly like `nfqws-arm`, `nfqws-aarch64`, etc. **Note:** The current `customize.sh` expects all binaries to be present in `zapret/` and renames the correct one to `nfqws` during install. So you should copy `nfqws-arm`, `nfqws-aarch64`, etc. into `module/zapret/`).
    - Place `dnscrypt-proxy` binaries in `module/dnscrypt/`.
    - Place `curl` binaries in `module/`.
    - Download `VpnHotspot.apk` to `module/system/app/`.
    - Download required lists (`ipset-v4.txt`, etc.) to `module/ipset/` and `module/list/`.

6.  **Zip it:**
    ```bash
    cd module
    7z a ../zapret-pocket.zip .
    ```
