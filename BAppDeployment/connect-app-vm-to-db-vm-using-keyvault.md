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
     - `db-password` → `SecureP@ssw0rd`

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
   - Example in Java:
     ```java
     import com.azure.identity.DefaultAzureCredentialBuilder;
     import com.azure.identity.DefaultAzureCredential;
     import com.azure.security.keyvault.secrets.SecretClient;
     import com.azure.security.keyvault.secrets.SecretClientBuilder;
     import com.azure.security.keyvault.secrets.models.KeyVaultSecret;


     public class KeyVaultExample {
         public static void main(String[] args) {
              // Replace with your Key Vault URL
              String keyVaultUrl = String keyVaultUri = "https://kvtvmdb1nprdne.vault.azure.net/";


             // Create a credential using DefaultAzureCredential
             DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();

             // 1. Establish the client using Managed Identity (DefaultAzureCredential)
             SecretClient secretClient = new SecretClientBuilder()
                 .vaultUrl(keyVaultUri)
                 .credential(credential) // Uses Managed Identity automatically
                 .build();

             // 2.0 Retrieve the secret values
             String dbName = secretClient.getSecret("db-name").getValue();
             String dbUser = secretClient.getSecret("db-username").getValue();
             String dbPassword = secretClient.getSecret("db-password").getValue();
   
             // 2.1 Print them (for demo only — don’t log secrets in production!)
             System.out.println("DB User: " + dbUser);
             System.out.println("DB Pass: " + dbPass);
             System.out.println("DB Name: " + dbName);


             // 3. Construct the final JDBC URL and connection (Step 2)
             String finalJdbcUrl = dbName + "?user=" + dbUser + "&password=" + dbPassword + "&sslmode=require";

         }
     }
     
     ```

   - No credentials are hardcoded — the VM identity handles auth.

     // Now use 'finalJdbcUrl', 'dbUser', and 'dbPassword' to establish the connection...
  
     ```bash
     jdbc:postgresql:"//azure-db-postgres-bea-prod-beyn-prod-ne-100.postgres.database.azure.com:5432/" + dbName + "?user=" + dbUser + "&password=" + dbPassword + "&sslmode=require";
     ```


Le resultat ConnectionString final JDBC sera comme suit:

```bash
jdbc:postgresql://azure-db-postgres-bea-prod-beyn-prod-ne-100.postgres.database.azure.com:5432/bea-international?user=bea_user&password={your_password}&sslmode=require
```
---


## How the App VM Connects to the DB VM

1. **Networking**
   - Ensure the **App Subnet** can reach the **DB Subnet** (via NSG rules, UDRs, and no firewall blocks).
   - Typically, allow inbound DB port (e.g., 1433 for SQL Server, 3306 for MySQL, 5432 for PostgreSQL) from the App Subnet.

---


