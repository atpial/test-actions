# test-actions
tests ci/cd using github actions

# Configuring WinRM Listener for HTTPS on EC2

This guide provides step-by-step instructions for setting up a WinRM listener on HTTPS for a Windows EC2 instance. This configuration allows for secure remote management.

## Prerequisites

- Windows EC2 instance
- Administrator access to the EC2 instance
- AWS security group configured to allow inbound traffic on port 5986

## Steps

### 1. Check Existing WinRM Listeners

Open PowerShell as an Administrator and run the following command to check existing WinRM listeners:

```powershell
winrm e winrm/config/listener
```

### 2. Create a Self-Signed Certificate

Generate a self-signed certificate with the DNS name set to your instance's public IP address or a domain name if available.

```powershell 
$cert = New-SelfSignedCertificate -DnsName "<IP_ADDRESS_OR_DOMAIN>" -CertStoreLocation Cert:\LocalMachine\My
```

### 3. Create a WinRM HTTPS Listener

Use the thumbprint from the newly created certificate to create a WinRM listener on HTTPS. Replace <IP_ADDRESS> and <COPIED_CERTIFICATE_THUMBPRINT> with the appropriate values.

```powershell 
$thumbprint = $cert.Thumbprint
winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="<IP_ADDRESS>"; CertificateThumbprint="$thumbprint"}'
```

### 4. Open Firewall Port for WinRM

Add a firewall rule to allow inbound traffic on port 5986.

```powershell 
$port = 5986
netsh advfirewall firewall add rule name="Windows Remote Management (HTTPS-In)" dir=in action=allow protocol=TCP localport=$port
```

### 5. Verify HTTPS Listener

Run the following command to verify that the HTTPS listener has been added:

```powershell 
winrm e winrm/config/listener
```
You should see an HTTPS listener with the correct configuration.

### 6. Update Security Group

Ensure your EC2 instance's security group allows inbound traffic on port 5986 and port 80. Add the following rule:

- Type: WinRM-HTTPS
- Protocol: TCP
- Port: 5986
- Source: Custom (e.g., 0.0.0.0/0 or a specific IP range)

and

- Type: HTTP
- Protocol: TCP
- Port: 80
- Source: Custom (e.g., 0.0.0.0/0 or a specific IP range)
