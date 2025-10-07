This is article explains how to connect the app vm to the db vm in Azure, where you want to keep secrets out of code and configs, and instead centralize them in **Key Vault**.

---

## How the App VM Gets DB Credentials from Key Vault

1. **Enable Managed Identity on the App VM**
   - Go to your App VM in the Azure portal.
   - Under **Identity**, enable **System-assigned managed identity** (or attach a User-assigned one).
   - This gives the VM an Azure AD identity that can be used to authenticate without storing credentials.

2. **Grant Key Vault Access**
   - In your Key Vault, add an **Access Policy** (or RBAC role if using Azure RBAC for Key Vault).
   - Assign the VM’s managed identity **Key Vault Secrets User** (or `get`/`list` permissions on secrets).
   - This ensures only the App VM can fetch the DB credentials.

3. **Store DB Credentials in Key Vault**
   - Store the database **name**, **username**, and **password** as secrets in Key Vault.
   - Example:
     - `db-name` → `mydb`
     - `db-username` → `dbadmin`
     - `db-password` → `SuperSecureP@ssw0rd`

4. **App VM Retrieves Secrets Securely**
   - Inside the App VM, your application (Python, .NET, Java, etc.) uses the **Azure Identity SDK** to authenticate via the VM’s managed identity.
   - 
   - Example in Python:
     ```python
     from azure.identity import DefaultAzureCredential
     from azure.keyvault.secrets import SecretClient

     key_vault_url = "https://<your-keyvault-name>.vault.azure.net/"
     credential = DefaultAzureCredential()
     client = SecretClient(vault_url=key_vault_url, credential=credential)

     db_user = client.get_secret("db-username").value
     db_pass = client.get_secret("db-password").value
     db_name = client.get_secret("db-name").value
     ```

   - 
   - Example in Python:
     ```java
     import com.azure.identity.DefaultAzureCredentialBuilder;
     import com.azure.identity.DefaultAzureCredential;
     import com.azure.security.keyvault.secrets.SecretClient;
     import com.azure.security.keyvault.secrets.SecretClientBuilder;
     import com.azure.security.keyvault.secrets.models.KeyVaultSecret;

     public class KeyVaultExample {
         public static void main(String[] args) {
              // Replace with your Key Vault URL
              String keyVaultUrl = "https://<your-keyvault-name>.vault.azure.net/";

        // Create a credential using DefaultAzureCredential
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();

        // Build the SecretClient
        SecretClient client = new SecretClientBuilder()
                .vaultUrl(keyVaultUrl)
                .credential(credential)
                .buildClient();

        // Retrieve secrets
        KeyVaultSecret dbUserSecret = client.getSecret("db-username");
        KeyVaultSecret dbPassSecret = client.getSecret("db-password");
        KeyVaultSecret dbNameSecret = client.getSecret("db-name");

        String dbUser = dbUserSecret.getValue();
        String dbPass = dbPassSecret.getValue();
        String dbName = dbNameSecret.getValue();

        // Print them (for demo only — don’t log secrets in production!)
        System.out.println("DB User: " + dbUser);
        System.out.println("DB Pass: " + dbPass);
        System.out.println("DB Name: " + dbName);
        }
      }
     ```

   - No credentials are hardcoded — the VM identity handles auth.

---

## How the App VM Connects to the DB VM

1. **Networking**
   - Ensure the **App Subnet** can reach the **DB Subnet** (via NSG rules, UDRs, and no firewall blocks).
   - Typically, allow inbound DB port (e.g., 1433 for SQL Server, 3306 for MySQL, 5432 for PostgreSQL) from the App Subnet.

2. **Database Authentication**
   - The app uses the retrieved credentials to connect:
     ```python
     import pyodbc

     conn_str = f"DRIVER={{ODBC Driver 17 for SQL Server}};SERVER={db_vm_private_ip};DATABASE={db_name};UID={db_user};PWD={db_pass}"
     conn = pyodbc.connect(conn_str)
     ```

     ```java
     conn_str = f"DRIVER={{ODBC Driver 17 for SQL Server}};SERVER={db_vm_private_ip};DATABASE={db_name};UID={db_user};PWD={db_pass}"      conn = pyodbc.connect(conn_str)
     ```
   - Replace with the appropriate driver for your DB engine.

---


Would you like me to sketch this as a **diagram** (App VM → Key Vault → DB VM) so you can use it in your documentation?
