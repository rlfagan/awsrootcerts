# AWS Root Certificate Installation Scripts

## Overview
These scripts are designed to download and install the Amazon Root CA certificates for Linux, Windows, and macOS systems. They resolve certificate trust issues when connecting to AWS-hosted services, such as FOSSA SaaS, where the root certificates are not already installed on the system.

## Script Usage

### Linux

1. Save the script as `install_amazon_certs.sh`.
2. Make the script executable:
   ```bash
   chmod +x install_amazon_certs.sh
   ```
3. Run the script with superuser privileges:
   ```bash
   sudo ./install_amazon_certs.sh
   ```

### Windows

1. Save the script as `Install-AmazonCerts.ps1`.
2. Open PowerShell as an Administrator.
3. Change the execution policy to allow the script to run:
   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force
   ```
4. Run the script:
   ```powershell
   .\Install-AmazonCerts.ps1
   ```

### macOS

1. Save the script as `install_amazon_certs_macos.sh`.
2. Make the script executable:
   ```bash
   chmod +x install_amazon_certs_macos.sh
   ```
3. Run the script with superuser privileges:
   ```bash
   sudo ./install_amazon_certs_macos.sh
   ```

## Script Details

### Linux Script (Bash)
This script downloads the Amazon Root CA certificates and installs them into the `/usr/local/share/ca-certificates` directory. It then updates the CA store.

```bash
#!/bin/bash

declare -A aws_certs=(
  ["AmazonRootCA1"]="https://www.amazontrust.com/repository/AmazonRootCA1.pem"
  ["AmazonRootCA2"]="https://www.amazontrust.com/repository/AmazonRootCA2.pem"
  ["AmazonRootCA3"]="https://www.amazontrust.com/repository/AmazonRootCA3.pem"
  ["AmazonRootCA4"]="https://www.amazontrust.com/repository/AmazonRootCA4.pem"
)

mkdir -p /usr/local/share/ca-certificates/aws

for cert in "${!aws_certs[@]}"; do
  echo "Downloading ${cert}..."
  curl -sSL -o "/usr/local/share/ca-certificates/aws/${cert}.crt" "${aws_certs[$cert]}"
done

update-ca-certificates

echo "AWS Root CA certificates have been installed successfully on Linux."
```

### Windows Script (PowerShell)
This PowerShell script downloads the AWS Root CA certificates and installs them in the "Trusted Root Certification Authorities" store for the local machine.

```powershell
$awsCertUrls = @{
    "AmazonRootCA1" = "https://www.amazontrust.com/repository/AmazonRootCA1.pem"
    "AmazonRootCA2" = "https://www.amazontrust.com/repository/AmazonRootCA2.pem"
    "AmazonRootCA3" = "https://www.amazontrust.com/repository/AmazonRootCA3.pem"
    "AmazonRootCA4" = "https://www.amazontrust.com/repository/AmazonRootCA4.pem"
}

foreach ($certName in $awsCertUrls.Keys) {
    $certPath = "$env:TEMP\$certName.cer"
    
    Write-Output "Downloading $certName..."
    Invoke-WebRequest -Uri $awsCertUrls[$certName] -OutFile $certPath
    
    Write-Output "Importing $certName into Trusted Root Certification Authorities..."
    Import-Certificate -FilePath $certPath -CertStoreLocation Cert:\LocalMachine\Root -Verbose
}

Write-Output "AWS Root CA certificates have been installed successfully on Windows."
```

### macOS Script (Bash)
This script downloads the AWS Root CA certificates and installs them in the System Keychain, making them trusted by all users and applications on the system.

```bash
#!/bin/bash

declare -A aws_certs=(
  ["AmazonRootCA1"]="https://www.amazontrust.com/repository/AmazonRootCA1.pem"
  ["AmazonRootCA2"]="https://www.amazontrust.com/repository/AmazonRootCA2.pem"
  ["AmazonRootCA3"]="https://www.amazontrust.com/repository/AmazonRootCA3.pem"
  ["AmazonRootCA4"]="https://www.amazontrust.com/repository/AmazonRootCA4.pem"
)

mkdir -p /tmp/aws-certs

for cert in "${!aws_certs[@]}"; do
  echo "Downloading ${cert}..."
  curl -sSL -o "/tmp/aws-certs/${cert}.pem" "${aws_certs[$cert]}"
done

for certFile in /tmp/aws-certs/*.pem; do
  sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain "$certFile"
done

rm -rf /tmp/aws-certs

echo "AWS Root CA certificates have been installed successfully on macOS."
```

## Troubleshooting

1. **Certificate Not Found**: If you encounter issues such as "certificate not found," ensure that the certificate URLs are accessible from your network.
2. **Permission Issues**: For Linux and macOS, run the script with `sudo` to install certificates at the system level.
3. **Invalid Certificate Format**: Ensure the certificates are downloaded as `.pem` or `.crt` files, as required by the platform.


