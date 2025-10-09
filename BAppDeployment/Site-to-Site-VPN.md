Deploying a **Site-to-Site (S2S) VPN** in Azure connects your on-premises network to an Azure Virtual Network (VNet) over an encrypted IPsec/IKE VPN tunnel.

---

## Prerequisites (On-Premises)

Before starting the Azure configuration, ensure you have the following information and a compatible on-premises device:

* A **compatible on-premises VPN device** (router or firewall).
* The **public IPv4 address** of your on-premises VPN device.
* The **address ranges/subnets** of your on-premises network (e.g., 192.168.1.0/24). These must **not overlap** with your Azure VNet address space.

---

##  Site-to-Site (S2S) VPN Deployment

### 1- Add a Gateway Subnet

The VPN Gateway requires a dedicated subnet named `GatewaySubnet`.

1.  Navigate to the **vnet-bea-nonprod-beyn-nonprod-ne-100** Virtual Network.
2.  In the left menu, select **Subnets**.
3.  Click **+ Gateway subnet** at the top. The name will auto-populate as **`GatewaySubnet`**.
4.  Specify the **10.5.11.0/26** Subnet address range.
5.  Click **Save**.

### 2- Create the Azure Virtual Network Gateway (VNG)

The VNG is the Azure side of the VPN connection.

1.  Search for **Virtual network gateways** and click **Create**.
2.  Configure the following settings on the **Basics** tab:
    * **Subscription** and **Resource Group**: Select the same ones as your VNet.
    * **Region**: Select the same region as your VNet.
    * **Name**: Enter a name for your gateway: vng-bea-nonprod-beyn-nonprod-ne-101
    * **Gateway type**: Select **VPN**.
    * **SKU**: Select the appropriate SKU: **VpnGw2AZ**.
    * **Virtual network**: Select the VNet and the **GatewaySubnet** subnet .
    * **Public IP address**: Select **Create new**, enter a **vgw-pub-ip-bea-nonprod-beyn-nonprod-ne-100** as the name, and choose the **Standard** SKU.
    * **Enable active-active mode**: Leave as **Disabled** (for a single S2S tunnel).
    * **Configure BGP**: Leave as **Disabled**, unless BGP routing is used.
3.  Click **Review + create**, then **Create**. *This deployment can take 30 to 45 minutes to complete.*

### 3- Create the Local Network Gateway (LNG)

The LNG is the Azure representation of your on-premises VPN device.

1.  Search for **Local network gateways** and click **Create**.
2.  Configure the following settings:
    * **Subscription**, **Resource Group**, and **Region**: Select the appropriate settings.
    * **Name**: Enter a name for your on-premises site (e.g., `OnPremises-Site`).
    * **Endpoint**: Select **IP address**.
    * **IP address**: Enter the **public IPv4 address** of your on-premises VPN device.
    * **Address space**: Enter the **address ranges** of your on-premises network (e.g., 192.168.1.0/24).
    * **Configure BGP settings**: Leave as **No** (unless you enabled BGP on the VNG).
3.  Click **Review + create**, then **Create**.

### 4- Create the VPN Connection

This step connects the Virtual Network Gateway (VNG) to the Local Network Gateway (LNG).

1.  Navigate to your **Virtual Network Gateway** (VNG) from Step 3.
2.  In the left menu, select **Connections**, and then click **+ Add**.
3.  Configure the connection settings:
    * **Name**: Enter a name for the connection (e.g., `Azure-to-OnPrem`).
    * **Connection type**: Select **Site-to-site (IPsec)**.
    * **Virtual network gateway**: Your VNG should be pre-selected.
    * **Local network gateway**: Select the **LNG** you created in Step 4.
    * **Shared key (PSK)**: Enter a **Pre-Shared Key** (a secret string, e.g., `MyS2SSecretKey123`). You will need to use this exact key on your on-premises VPN device.
    * **IKE Protocol**: Select the version compatible with your on-premises device (usually **IKEv2**).
4.  Click **OK** to create the connection.

---

## 5- Configure Your On-Premises VPN Device

The final critical step is configuring your physical, on-premises VPN device (router or firewall) to match the Azure gateway settings.

1.  Access the configuration interface for your on-premises VPN device.
2.  Configure an **IPsec VPN tunnel** with the following parameters:
    * **Remote Peer/Gateway IP**: The **Public IP address** of your **Azure Virtual Network Gateway** (you can find this on the VNG's Overview page).
    * **Local Peer/Gateway IP**: The **public IPv4 address** of your on-premises device (used in the LNG).
    * **Remote Network/Subnets**: The **address space** of your Azure VNet (e.g., 10.0.0.0/16).
    * **Local Network/Subnets**: The **address space** of your on-premises network (e.g., 192.168.1.0/24).
    * **Pre-Shared Key (PSK)**: The **exact shared key** entered in Step 5.
    * **IKE/IPsec Policies**: Match the IKE Phase 1 and Phase 2 encryption/integrity settings, Diffe-Hellman groups, and lifetime values to the **Azure VPN Gateway default policies** or a custom policy you defined.
3.  Save and activate the configuration on your device.

---

## 6- Verify the Connection

The tunnel status should change once the on-premises device is configured and the connection is established.

1.  In the Azure portal, navigate to your **Virtual Network Gateway**.
2.  Select **Connections**.
3.  The connection you created should eventually show a **Status** of **Connected**.
4.  **Test connectivity** by initiating traffic (like a **ping**) from a host in your on-premises network to an Azure Virtual Machine, or vice versa. This is often necessary to *bring up* the VPN tunnel.
