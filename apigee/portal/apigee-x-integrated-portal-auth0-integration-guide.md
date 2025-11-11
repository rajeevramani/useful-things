# How to Configure Auth0 SAML SSO for Apigee X Integrated Developer Portal

## Overview

To configure your Apigee Integrated Portal with SAML and use Auth0 as your Identity Provider (IDP), you'll need to set up the SAML connection on both Apigee and Auth0 and exchange metadata between the two systems. This process enables single sign-on (SSO) for your portal users, authenticating them through Auth0 instead of Apigee's built-in IDP.

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
  - Go to: `Applications` → *Your Application* (or create a new Application for this integration)
  - Navigate to `Addons` → `SAML2 Web App`
  - **Enable** the SAML2 Web App Addon and open the **Settings** tab.

### Set the Application Callback URL

- In the SAML2 Web App settings, set the **Application Callback URL** to the **Apigee AssertionConsumerService (ACS) URL** that you copied from the SP metadata in Step 1.

### Configure the JSON Settings

In the JSON configuration section, set the following (replace placeholders as needed):

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

### Gather Auth0 Identity Provider Information

After saving the SAML2 Web App configuration, you'll need to copy the following values from Auth0 to configure Apigee:

- **Download the Auth0 signing certificate** (in PEM format) - you'll upload this to Apigee
- **Copy the Issuer** - this becomes the **IdP Entity ID** for Apigee
- **Copy the Identity Provider Login URL** - this becomes the **Sign-in URL** for Apigee

> **Note:** Auth0 expects the SAML `nameidentifier` to be an email address. The `nameIdentifierProbes` configuration above handles this by mapping the user's email to the NameID field. If you encounter issues, you may need to create a rule in Auth0 to explicitly map `nameid` to the user's email address.

## 3. Confirm/Set NameID Mapping

- **Verify** in Auth0 that the SAML `NameID` is set to the user's email and the format is as above. This is handled by the `"nameIdentifierFormat"` and `"nameIdentifierProbes"` configuration.

## 4. Complete the Apigee SAML Configuration

Now you'll finalize the integration in Apigee by entering the Auth0 Identity Provider details:

- In Apigee, navigate back to:
  `Publish` → `Portals` → *Your Portal* → `Accounts` → `Authentication` → `Identity Providers` → `SAML`
- Enter the following information from Auth0:
  - **Sign-in URL**: The Identity Provider Login URL from Auth0
  - **IdP Entity ID**: The Issuer from Auth0
  - **Certificate**: Upload the Auth0 signing certificate (PEM format) so that Apigee can validate Auth0's signed assertions
- **Save** the configuration.

## 5. Enable SAML, Save, and Test

- **Enable the SAML identity provider** in Apigee and **save** the configuration.
- In Auth0, confirm that the SAML connection is enabled and the Addon is active for your application.
- Navigate to your Apigee developer portal sign-in page. You should now see a **"Login with SAML"** option.
- **Test your integration** by clicking the "Login with SAML" button and authenticating through Auth0.
- Verify that users are provisioned correctly and attributes (email, first name, last name) are populated as expected.

## 6. Troubleshooting & Best Practices

- **ACS URL is critical:**
  If the assertion is not POSTed to the correct ACS URL, SSO will fail.
- **Attributes must be named exactly** as:
  `email`, `first_name`, `last_name`
- **NameID must be the user's email** in the correct format.
- After custom domain changes in Apigee, **re-download SP metadata and update Auth0** with any new/changed URLs.
- **Do not replay SAML assertions**—start each test/login fresh.
- **Debugging SSO errors:**
  - Use browser developer tools (Network tab) to inspect SAML requests and responses
  - Check Auth0 logs for authentication errors and SAML assertion details
  - Review Apigee portal logs for any validation errors

---

**By following this step-by-step guide, and using all the fields from Apigee SP metadata (especially the ACS URL), your SAML SSO integration will be robust and fully compliant with Apigee X requirements.**
