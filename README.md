# Azure Point-to-Site/Point-to-Point VPN Setup for AV Rack Monitoring

This project demonstrates how to set up a secure connection between Azure and AV racks to monitor equipment using a TP-Link ER605 router with IPsec VPN. The guide includes steps to create an Azure Virtual Network (VNet), configure a VPN Gateway, generate certificates for client devices, and establish secure communication with AV racks.

---

## Features
- **Azure VPN Gateway:** Connects Azure to your on-premises AV racks.
- **Point-to-Site/Point-to-Point VPN:** Options to connect individual devices or entire sites to Azure.
- **TP-Link ER605 Configuration:** Set up IPsec VPN for secure connectivity.
- **Certificate-Based Authentication:** Secure client access with self-signed certificates.

---

## Prerequisites
1. **Azure Subscription:** Ensure access to the Azure portal.
2. **TP-Link ER605 Router:** For IPsec VPN setup.
3. **Tools:**
   - OpenSSL (to generate certificates).
   - VPN client software for end-user devices.
4. **Permissions:** Admin access to your router and Azure account.

---

## Steps

### 1. Create an Azure Virtual Network (VNet)
```bash
# Log in to the Azure portal
# Navigate to Virtual Networks and create a new VNet

# Configuration
Name: AV-Rack-VNet
Address Space: 10.0.0.0/16
Subnet: 10.0.0.0/24

# CLI Example
az network vnet create \
  --name AV-Rack-VNet \
  --resource-group MyResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name GatewaySubnet \
  --subnet-prefix 10.0.0.0/24
```

### 2. Set Up an Azure VPN Gateway
```bash
# Go to VPN Gateways and create a new gateway

# Configuration
Gateway Type: VPN
VPN Type: Route-Based
SKU: Basic or VpnGw1
Virtual Network: AV-Rack-VNet

# CLI Example
az network vnet-gateway create \
  --name AV-VPN-Gateway \
  --public-ip-address AV-VPN-Gateway-IP \
  --resource-group MyResourceGroup \
  --vnet AV-Rack-VNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

# Note: Provisioning the gateway may take up to 45 minutes
```

### 3. Generate Certificates for Authentication
```bash
# Generate a self-signed root certificate
openssl req -x509 -nodes -newkey rsa:2048 -keyout RootCert.key -out RootCert.pem -days 365

# Export the root certificate to Base64 for Azure
openssl x509 -in RootCert.pem -outform DER | base64 > RootCertBase64.txt

# Generate client certificates signed by the root
openssl genrsa -out ClientCert.key 2048
openssl req -new -key ClientCert.key -out ClientCert.csr
openssl x509 -req -in ClientCert.csr -CA RootCert.pem -CAkey RootCert.key -CAcreateserial -out ClientCert.pem -days 365

# Convert the client certificate to a PFX file for import
openssl pkcs12 -export -out ClientCert.pfx -inkey ClientCert.key -in ClientCert.pem
```

### 4. Configure the TP-Link ER605 Router
```bash
# Log in to the TP-Link ER605 admin interface
# Navigate to VPN > IPsec and create a new IPsec VPN profile

# Configuration
Mode: Aggressive or Main
Authentication: Pre-Shared Key or Certificate-Based
Encryption: AES-256
Local/Remote Subnets: Match your Azure VNet and on-premises subnets

# Example Pre-Shared Key: Ensure it matches the key configured in Azure
Pre-Shared Key: MyStrongPreSharedKey123

# Save the configuration and enable the IPsec tunnel
```

### 5. Connect Client Devices
```bash
# Install a VPN client (e.g., Azure VPN Client or native OS VPN tools)

# Import the client certificate into the device
# Windows Example
certutil -importpfx ClientCert.pfx

# Configure the VPN client with:
- Gateway Public IP: Obtain from the Azure VPN Gateway
- Connection Type: IPsec/IKEv2
- Authentication: Certificate-Based

# Test the connection to ensure it is successful
```

### 6. Test Connectivity
```bash
# Establish the VPN connection from your device or TP-Link ER605 router
# Verify access to devices in the Azure VNet and on-premises AV racks

# Ping a resource in the Azure VNet to confirm connectivity
ping 10.0.0.5

# Use monitoring tools (e.g., QSC MP-M40 software) to check equipment status
```

---

## Notes
- **Cost Consideration:** Be aware of Azure VPN Gateway pricing based on SKU and data transfer.
- **Security:** Regularly update certificates and ensure strong passwords for the VPN.
- **Troubleshooting:**
  - Check logs on the TP-Link router and Azure VPN Gateway for connection issues.
  - Verify that subnet ranges do not overlap.

---

## Future Enhancements
- Automate certificate renewal and deployment.
- Implement monitoring dashboards using Azure Monitor.
- Explore additional configurations for high availability (HA) setups.
