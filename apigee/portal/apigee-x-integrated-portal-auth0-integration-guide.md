# Configuring Auth0 as SAML Identity Provider for Apigee X Integrated Developer Portal

## 1. Gather Required Info From Apigee

- **Log in to Apigee Cloud Console**
- Go to
  `Publish` → `Portals` → *Your Portal* → `Accounts` → `Authentication`
- Select the **SAML** identity provider type.
- Download or view the **Service Provider (SP) metadata**.
  Note the following:
  - **Assertion Consumer Service (ACS) URL**
  - **Entity ID**
  - **(Optional) Certificate/public key**

## 2. Create & Configure SAML Connection in Auth0

- In Auth0, navigate to
  `Applications` → *Your Application* → `Addons` → `SAML2 Web App`
- Enable the Addon and go to the **Settings** tab.

#### Use this JSON configuration (replace placeholders with Apigee metadata):

```json
{
  "nameIdentifierFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress",
  "nameIdentifierProbes": [ "email" ],
  "mappings": {
    "email": "email",
    "first_name": "given_name",
    "last_name": "family_name"
  },
  "audience": "REPLACE_WITH_APIGEE_ENTITY_ID",
  "recipient": "REPLACE_WITH_APIGEE_ACS_URL",
  "destination": "REPLACE_WITH_APIGEE_ACS_URL"
}
```
- `nameIdentifierFormat` sets the NameID format for Apigee.
- `nameIdentifierProbes` ensures the user's email is used for NameID.
- `mappings` guarantees the required attributes (`email`, `first_name`, `last_name`) are present and correctly mapped.
- Replace `audience/destination/recipient` with the actual Entity ID and ACS URL from Apigee's metadata.

## 3. (Optional) Add Additional Attribute Mappings

If other attributes are needed, extend the `mappings` block in the Auth0 JSON accordingly.

## 4. Upload Auth0 Signing Certificate to Apigee

- Download Auth0's signing certificate (PEM format) from the SAML2 Addon settings.
- In Apigee portal SAML settings, upload the IdP certificate.

## 5. Finalize & Test the Integration

- Enable and **save** the SAML provider in Apigee.
- Ensure the SAML connection is **enabled** in Auth0 for your application.
- Test SSO logins via your Apigee portal.
- Validate user provisioning and mappings (`email`, `first_name`, `last_name`).

## 6. Troubleshooting Tips

- Make sure the SAML assertion contains attributes with names **exactly** as: `email`, `first_name`, `last_name`.
- NameID **must** match the user's email and use the right format.
- Review Apigee and Auth0 logs for errors and attribute mapping problems.
- If using custom domains on Apigee, update Auth0 Post-SP metadata changes.
- Never replay SAML assertions for multiple login attempts.

---

**With the above steps and JSON settings, you'll have a robust SAML SSO integration between Auth0 and Apigee X's developer portal.**
