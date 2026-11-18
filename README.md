# QuakeC Studio Packages

APT and YUM/DNF repositories for [QuakeC Studio](https://www.quakecstudio.com), served at [packages.quakecstudio.com](https://packages.quakecstudio.com).

| Package | Types | Architectures |
|---------|-------|---------------|
| [`quakec-studio`](https://github.com/quakecstudio/editor) | `.deb`, `.rpm` | amd64/x86_64, arm64/aarch64 |

## Installation

### APT (Debian / Ubuntu)

```bash
curl -fsSL https://packages.quakecstudio.com/gpg/quakecstudio.asc \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/quakecstudio-archive-keyring.gpg > /dev/null

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/quakecstudio-archive-keyring.gpg] https://packages.quakecstudio.com/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/quakecstudio.list > /dev/null

sudo apt-get update && sudo apt-get install -y quakec-studio
```

### YUM / DNF (Fedora / RHEL)

```bash
sudo tee /etc/yum.repos.d/quakecstudio.repo << 'EOF'
[quakecstudio]
name=QuakeC Studio
baseurl=https://packages.quakecstudio.com/rpm/$basearch/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.quakecstudio.com/gpg/quakecstudio.asc
enabled=1
EOF

sudo dnf install -y quakec-studio
```

## GPG key

RPM packages and all repository metadata are signed with [`gpg/quakecstudio.asc`](gpg/quakecstudio.asc), an Ed25519 key with fingerprint `B00D 6EBB BD27 0E2A 6C12 F473 0320 6F03 F365 A966`. The `environments/github` root of [quakecstudio/infra](https://github.com/quakecstudio/infra) generates the key and sets its private half and passphrase as the `GPG_PRIVATE_KEY` and `GPG_PASSPHRASE` Actions secrets on quakecstudio/editor.

## How releases land here

Nothing in this repository builds packages. The editor's release workflow builds and signs its `.deb` and `.rpm` files, then checks out this repository with a deploy key (`PACKAGES_DEPLOY_KEY`, write access to this repository only) and:

1. copies `.deb` files into `apt/pool/main/` and `.rpm` files into `rpm/x86_64/` or `rpm/aarch64/`
2. regenerates `apt/dists/stable/` with `dpkg-scanpackages` and `apt-ftparchive release -c apt-ftparchive.conf`, then signs `Release` into `Release.gpg` and `InRelease`
3. runs `createrepo_c --update` for each RPM architecture and signs `repodata/repomd.xml`
4. commits `quakec-studio <version>` and pushes to `main`

Because that push uses a deploy key rather than `GITHUB_TOKEN`, it triggers [`publish-to-s3.yml`](.github/workflows/publish-to-s3.yml), which syncs the repository to S3 and invalidates CloudFront.

## Layout

```
apt/pool/main/          .deb packages
apt/dists/stable/       APT metadata (Release, InRelease, Packages)
rpm/x86_64/             x86_64 .rpm packages and repodata/
rpm/aarch64/            aarch64 .rpm packages and repodata/
gpg/quakecstudio.asc    public signing key
apt-ftparchive.conf     Release file fields
index.html              landing page
```
