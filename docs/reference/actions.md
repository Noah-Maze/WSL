(reference::actions)=
# GitHub actions

<!-- TODO: intro paragraph  -->

## download-rootfs
Download (and cache) the latest Rootfs tarball for a particular release of Ubuntu WSL.

### Arguments
- `distros`: a comma-separated list of distros to download. Use the names as shown in WSL. Defaults to `Ubuntu`. See more: [Ubuntu WSL distributions](reference::distros).
- `path`: the path where to store the tarball. The tarball will end up as `${path}\${distro}.tar.gz`. PowerShell-style environment variables will be expanded. If there already exists a tarball at the download path, a checksum comparison will be made to possibly skip the download.

### Example
```yaml
 - name: Download Jammy rootfs
   uses: Ubuntu/WSL/.github/actions/download-rootfs@main
   with:
    distro: Ubuntu-22.04
    path: '${env:UserProfile}\Downloads\rootfs'
```


## wsl-bash
Run arbitrary bash code in your distro.

### Arguments
 - `distro`: an installed WSL distro. Write its name as it would appear on WSL. Read more: [Ubuntu WSL distributions](reference::distros)
 - `exec`: the script to run.
 - `working-dir`: path to the WSL directory to run the script in. Set to `~` by default.

### Example
```yaml
 - name: Install pip
   uses: Ubuntu/WSL/.github/actions/wsl-bash@main
   with:
    distro: Ubuntu-20.04
    working-dir: /tmp/github/
    exec: |
        DEBIAN_FRONTEND=noninteractive sudo apt update
        DEBIAN_FRONTEND=noninteractive sudo apt install python3-pip
```

## wsl-checkout
This action checks out your repository in a WSL distro. If you want to check it out on the Windows file system, use the regular `actions/checkout` action instead.

### Arguments
- `distro`: an installed WSL distro. Write its name as it would appear on WSL. Read more: [Ubuntu WSL distributions](reference::distros)
- `working-dir`: the path where the repository should be cloned. Set to `~` by default.
- `submodules:`: Whether to fetch sub-modules or not. False by default.

### Example
```yaml
 - name: Check out the repository
   uses: Ubuntu/WSL/.github/actions/wsl-checkout@main
   with:
    distro: Ubuntu-20.04
    working-dir: /tmp/github/
    submodules: true
```

## wsl-install
Install the Windows Subsystem for Linux application, and optionally an Ubuntu WSL application.

### Arguments
- `distro`: Optional argument
  - Blank (default): don't install any Ubuntu WSL distro
  - Distro name: any of the available distros in the Microsoft store. Write its name as shown in WSL. Read more: [Ubuntu WSL distributions](reference::distros)

### Example
```yaml
 - name: Install or update WSL
   uses: Ubuntu/WSL/.github/actions/wsl-install@main
   with:
    distro: Ubuntu-20.04
```
