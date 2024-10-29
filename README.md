### **Introduction**

WSO2 Identity Server (WSO2IS) provides a comprehensive solution for managing user identities and enabling secure single sign-on (SSO) across applications. In this guide, we will configure WSO2IS as the central SAML identity provider (IdP) for Wazuh, an open-source security monitoring platform. Integrating WSO2IS with Wazuh via SAML-based SSO simplifies the login process, enhances security, and allows users to access Wazuh with a single set of credentials managed centrally in WSO2IS.

### **Objectives**

The primary objectives of this integration are:
1. **Enable SSO**: Facilitate a seamless login experience by configuring WSO2IS as the SAML IdP for Wazuh.
2. **Centralized User Management**: Leverage WSO2IS to manage user credentials, roles, and access rights across applications.
3. **Enhanced Security**: Improve security by configuring SAML with proper certificate-based encryption and secure SSO.

### **Security Requirements**

1. **Secure Communication**: Use HTTPS for all communications between Wazuh and WSO2IS to protect data in transit.
2. **Certificate Management**: Ensure SAML responses are signed with a secure certificate, enforcing authenticity and integrity.
3. **Role-based Access Control**: Map WSO2IS roles to Wazuh roles, granting permissions as per organizational policies.
4. **Session Management**: Properly configure session timeouts and logout URLs to manage user sessions securely.

---

![Y27AjCz1z4BachcF4XVWX](https://github.com/user-attachments/assets/db69cf12-cc1d-4755-b35d-5ab912bab57e)

### **Configuration Steps**

#### **Prerequisites**

- A running instance of WSO2 Identity Server (WSO2IS).
- Wazuh deployed on a server, accessible via HTTPS.
- Administrator credentials for both WSO2IS and Wazuh.

#### **Step 1: Configure WSO2 Identity Server for SAML SSO**

1. **Access WSO2IS Management Console:**
   - Open a browser and go to `https://<your_wso2_domain>:9443/carbon`.
   - Log in with an administrator account.

2. **Register Wazuh as a Service Provider (SP):**
   - In the left-hand menu, navigate to **Main > Identity > Service Providers**.
   - Click **Add** and name your Service Provider (e.g., "Wazuh SSO").
   - Click **Register** to proceed to the service provider’s configuration page.

3. **Configure SAML2 Web SSO:**
   - In the **Inbound Authentication Configuration** section, click **SAML2 Web SSO Configuration** > **Configure**.
   - Fill out the following fields:

     - **Issuer**: Use a unique identifier for Wazuh, such as `wazuh-sso`.
     - **Assertion Consumer URL**: Set this to the Wazuh SAML endpoint:
       ```
       https://<your_wazuh_domain>/auth/realms/<realm_name>/broker/wso2is/endpoint
       ```
       Replace `<your_wazuh_domain>` with your Wazuh domain or IP and `<realm_name>` with your realm name.
     - **Service Provider Entity ID**: Use:
       ```
       https://<your_wazuh_domain>/auth/realms/<realm_name>
       ```
     - **Enable Response Signing** and **Enable Assertion Signing** for security.
     - **Enable Single Logout** if you want centralized session termination.
     - **Logout URL**: Specify the URL where users should be redirected on logout:
       ```
       https://<your_wazuh_domain>/auth/realms/<realm_name>/protocol/openid-connect/logout
       ```
   - Click **Update** to save the SAML configuration.

4. **Download the SAML Metadata Certificate:**
   - In the **Certificates** section, download the **X.509 Certificate**. Save it as `wso2is_certificate.pem` for later use in Wazuh.

#### **Step 2: Configure Wazuh for SAML SSO with WSO2IS**

1. **Edit Wazuh Configuration File:**
   - Open the Wazuh configuration file on your server:
     ```bash
     sudo nano /etc/wazuh/wazuh.yaml
     ```
   - Locate the `auth` section and configure SAML settings as follows:

     ```yaml
     auth:
       saml:
         enabled: true
         entity_id: "https://<your_wazuh_domain>/auth/realms/<realm_name>"
         sso_url: "https://<your_wso2_domain>:9443/samlsso"
         certificate: "/etc/wazuh/wso2is_certificate.pem"
     ```
   - Replace:
     - `<your_wazuh_domain>` and `<realm_name>` with your Wazuh’s domain and realm.
     - `<your_wso2_domain>` with your WSO2IS server’s domain or IP.

2. **Upload and Secure the Certificate:**
   - Place the downloaded WSO2IS certificate (`wso2is_certificate.pem`) in `/etc/wazuh/`.

     ```bash
     sudo cp /path/to/downloaded/wso2is_certificate.pem /etc/wazuh/
     sudo chown wazuh:wazuh /etc/wazuh/wso2is_certificate.pem
     sudo chmod 600 /etc/wazuh/wso2is_certificate.pem
     ```

3. **Restart Wazuh to Apply Changes:**
   ```bash
   sudo systemctl restart wazuh-manager
   ```

#### **Step 3: Assign Users and Test SSO**

1. **Assign Users to the Wazuh Application in WSO2IS:**
   - In WSO2IS, navigate to **Main > Identity > Users and Roles**.
   - Assign users to roles associated with the Wazuh service provider.

2. **Test the SSO Configuration:**
   - Open Wazuh’s SSO login URL:
     ```
     https://<your_wazuh_domain>/auth/realms/<realm_name>/protocol/saml
     ```
   - Log in with a WSO2IS account assigned to Wazuh. If configured correctly, users will be redirected to WSO2IS, authenticated, and redirected back to Wazuh.

3. **Troubleshoot if Necessary:**
   - Verify settings in WSO2IS and Wazuh if login fails.
   - Check WSO2IS and Wazuh logs for any SAML-related errors.

---

### **Optional Configurations**

1. **Role Mapping in WSO2IS:**
   - Navigate to **Main > Identity > Service Providers > Wazuh SSO > Claim Configuration**.
   - Map WSO2IS roles to Wazuh roles for granular access control.

2. **Session Management and Timeouts:**
   - Configure session durations and single logout behavior in both WSO2IS and Wazuh.

3. **Custom Attribute Mapping**:
   - Add custom claims in WSO2IS for additional user attributes if Wazuh requires them.

---

### **Conclusion**

Integrating WSO2 Identity Server with Wazuh through SAML-based SSO streamlines user management and enhances security. By centralizing authentication, users benefit from a single set of credentials, while administrators gain greater control over access policies. This setup, leveraging WSO2IS's robust identity management features, not only improves user experience but also strengthens organizational security with secure, certificate-based authentication and role-based access controls.
