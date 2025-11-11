# How to Configure Auth0 SAML SSO for Apigee X Integrated Developer Portal

## 1. Retrieve Service Provider (SP) Metadata from Apigee

- Log in to the **Apigee Cloud Console**.
- Navigate to:
  `Publish` → `Portals` → *Your Portal* → `Accounts` → `Authentication`
- **Select the SAML identity provider** option.
- Download or copy the **SP metadata file**.
- Extract the following from the SP metadata:
  - **Entity ID**: Uniquely identifies Apigee portal as a SAML service provider.
  - **Assertion Consumer Service (ACS) URL**: The endpoint where Auth0 should send SAML assertions.
  - **Certificate/public key** (if required for Auth0 to trust the SP; usually not needed unless mutual trust is set up)

> **Tip:**
> The ACS URL is typically found as the value of the `Location` property in the metadata under `<md:AssertionConsumerService ... />`.

## 2. Create and Configure SAML Connection in Auth0

- In the **Auth0 Dashboard**:
  - Go to: `Applications` → *Your Application* → `Addons` → `SAML2 Web App`
  - **Enable** the SAML2 Web App Addon, open the **Settings** tab.

### In the JSON configuration, set the following (replace placeholders as needed):

```json
{
  "audience":     "REPLACE_WITH_APIGEE_ENTITY_ID",
  "recipient":    "REPLACE_WITH_APIGEE_ACS_URL",
  "destination":  "REPLACE_WITH_APIGEE_ACS_URL",
  "nameIdentifierFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress",
  "nameIdentifierProbes": [ "email" ],
  "mappings": {
    "email": "email",
    "first_name": "given_name",
    "last_name": "family_name"
  }
}
```

- **audience**: Entity ID copied from the Apigee SP metadata.
- **recipient**: ACS URL copied from the Apigee SP metadata.
- **destination**: ACS URL copied from the Apigee SP metadata.
- **nameIdentifierFormat**: Always `"urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress"` for Apigee.
- **nameIdentifierProbes**: Ensure the user's email is placed as NameID.
- **mappings**: These must match Apigee's expected attribute names exactly.

## 3. Confirm/Set NameID Mapping

- **Verify** in Auth0 that the SAML `NameID` is set to the user's email and the format is as above. This is handled by the `"nameIdentifierFormat"` and `"nameIdentifierProbes"` configuration.

## 4. Upload the Auth0 Signing Certificate to Apigee

- **Download the Auth0 signing certificate** (in PEM format) from the SAML Addon settings, if not already present.
- In Apigee, go to your portal's SAML IdP settings and **upload this certificate** so that Apigee can validate Auth0's signed assertions.

## 5. Enable SAML, Save, and Test

- **Enable the SAML identity provider** in Apigee and **save** the configuration.
- In Auth0, confirm that the SAML connection is enabled and the Addon is active for your application.
- **Test your integration** by logging in to the Apigee developer portal using SSO.
- Make sure users are provisioned and attributes (email, first name, last name) are correctly populated.

## 6. Troubleshooting & Best Practices

- **ACS URL is critical:**
  If the assertion is not POSTed to the correct ACS URL, SSO will fail.
- **Attributes must be named exactly** as:
  `email`, `first_name`, `last_name`
- **NameID must be the user's email** in the correct format.
- After custom domain changes in Apigee, **re-download SP metadata and update Auth0** with any new/changed URLs.
- **Do not replay SAML assertions**—start each test/login fresh.

---

**By following this step-by-step guide, and using all the fields from Apigee SP metadata (especially the ACS URL), your SAML SSO integration will be robust and fully compliant with Apigee X requirements.**
