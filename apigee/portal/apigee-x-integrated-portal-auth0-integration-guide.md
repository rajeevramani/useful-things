# How to Configure Auth0 SAML SSO for Apigee X Integrated Developer Portal

To configure your Apigee Integrated Portal with SAML and use Auth0 as your Identity Provider (IDP), you'll need to set up the SAML connection on both Apigee and Auth0 and exchange metadata between the two systems. This process enables single sign-on (SSO) for your portal users, authenticating them through Auth0 instead of Apigee's built-in IDP.

### Steps for SAML Integration

#### Configure Service Provider (Apigee)

- In the Apigee UI, go to **Publish > Portals**.
- Select your portal and open the **Accounts** section.
- Under the **Authentication** tab, choose **Identity Providers > SAML** and enable it.
- Save the configuration, then access and download the Apigee SAML Service Provider (SP) metadata XML.
- From the metadata file, locate the `AssertionConsumerService` URL; you’ll need this for setting up Auth0.

#### Configure Identity Provider (Auth0)

- In Auth0, create a new Application for this integration under **Applications**.
- Go to **Add Ons** on that Application and enable **SAML2 WEB APP**.
- In the SAML2 WEB APP settings:
  - Set the **Application Callback URL** to the Apigee AssertionConsumerService URL.
  - Download the Auth0 certificate (PEM format) to be uploaded in Apigee’s SAML configuration.
  - The **Issuer** becomes the IdP entity ID and the **Identity Provider Login URL** becomes the sign-in URL for Apigee. Copy both values for Apigee's SAML setup.
  - Auth0 expects a SAML `nameidentifier` to be an email. You may need to create a rule in Auth0 to map `nameid` to the user’s email address.

#### In the JSON configuration of the setting popup, set the following (replace placeholders as needed)

```json
{
  "nameIdentifierFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress",
  "nameIdentifierProbes": [ "email" ],
  "mappings": {
    "email": "email",
    "first_name": "given_name",
    "last_name": "family_name"
  }
}
```

#### Finalize Integration in Apigee

- In the Apigee SAML setup, enter the Auth0 **Sign-in URL**, **IdP Entity ID**, and upload the Auth0 certificate.
- Save your configuration. You should now see a "Login with SAML" option on your developer portal sign-in page.

#### Testing and Troubleshooting

- Test the integration by signing in through the SAML option on your portal.
- Use browser developer tools and Auth0 logs if you need to debug any SSO errors during login.

### Tips and Notes

- If needed, you can disable Apigee’s built-in identity provider so only SAML authentication is used, but at least one provider must stay enabled for user sign-in.
- Refer to Auth0's dashboard for additional configuration options and metadata details, especially for custom domains or organization-specific SSO.
- Apigee documentation and Auth0's SAML setup guides offer further details for advanced scenarios.

This configuration allows you full control over user management, leverages centralized Auth0 identity, and enables seamless SSO across Apigegee portals.
