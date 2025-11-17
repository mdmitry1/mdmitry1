# OnlyOffice Desktop Editors Installation

This guide provides step-by-step instructions for installing OnlyOffice Desktop Editors on Debian-based Linux systems.

## Installation Steps

### 1. Run this command before installation in order to avoid GUI issues 
```
echo ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true | sudo debconf-set-selections
```

### 2. Create GPG Directory

First, create a GPG directory with appropriate permissions:

```bash
mkdir -p -m 700 ~/.gnupg
```

### 3. Import GPG Key

Download and import the OnlyOffice GPG key:

```bash
gpg --no-default-keyring --keyring gnupg-ring:/tmp/onlyoffice.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys CB2DE8E5
```

### 4. Set Keyring Permissions

Configure the keyring file with proper permissions and ownership:

```bash
chmod 644 /tmp/onlyoffice.gpg
sudo chown root:root /tmp/onlyoffice.gpg
sudo mv /tmp/onlyoffice.gpg /usr/share/keyrings/onlyoffice.gpg
```

### 5. Add Repository

Add the OnlyOffice repository to your system's sources list:

```bash
echo 'deb [signed-by=/usr/share/keyrings/onlyoffice.gpg] https://download.onlyoffice.com/repo/debian squeeze main' | sudo tee -a /etc/apt/sources.list.d/onlyoffice.list
```

### 5. Update and Install

Update your package list and install OnlyOffice:

```bash
sudo apt update
sudo apt-get install onlyoffice-desktopeditors
```

## Notes

- These instructions are for Debian-based distributions (Ubuntu, Linux Mint, etc.)
- Root or sudo privileges are required for installation
- The GPG key ensures package authenticity and security
