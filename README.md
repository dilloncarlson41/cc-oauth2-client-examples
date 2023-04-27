#### This repository includes step by step examples of how to connect clients to Confluent Cloud via OAuth2 (token based) and Azure AD. 

## Prerequisites
Access to an Azure AD account
Permissions to create app registrations and update the application manifest in Azure AD
Confluent Cloud environment


## Steps for setting up an application to use OAuth2 authentication and authorization via the Confluent Cloud Console and Azure Portal
 
1. Register a client application in Azure AD
Go to Azure AD (must have permissions to create an app registration and view/update an applications manifest)
Click ‘App registrations’  on the side menu, then click ‘New registration’
Give the app a name and select the first option under ‘Supported account types’ (this disallows other AD accounts from using the client application being registered)
Click Register

2. Register Azure AD as the Identity Provider (IDP) in Confluent Cloud
Go to the Confluent Cloud console
Click on the hamburger icon in upper right corner, then click on ‘Accounts & access’
Click on the ‘Identity Providers’ tab, then click on ‘+Add provider’, then click on the Azure AD tile, click ‘Next’ 
Enter an IDP name(descriptive of the IDP being registered), description, and tenant ID from the Azure AD instance being registered. The ‘Tenant ID’ can be found in the Azure AD portal on the ‘Overview’ page
Click on ‘Validate and save’

3. Setup identity pools and grant access
Go to the Confluent Cloud console
Click on the hamburger icon in upper right corner, then click on ‘Accounts & access’
Click on the ‘Identity Providers’ tab, click on the IDP registered in step 2, then click on ‘+Add pool’
Give your identity pool a name, description, and Identity claim (Identity claim is a key/value identifier in the JWT token. Claims are managed in Azure AD). For simplicity and demonstration purposes, enter ‘claims.sub’.
Click on the ‘Advanced’ toggle and enter ‘has(claims.sub)’. This is for simplicity and demonstration purposes. This identity pool will now allow clients that submit tokens with a ‘sub’ property to be allowed to the roles granted to this pool. 
Click ‘Next’
Click ‘Add new permissions’
If there is an existing service account or user to mirror authorization for, click Import permissions. Otherwise, pick the roles which this identity pool should posses
Click ‘Next’, review the permissions, click ‘Validate and save’

4. Enter credentials and properties in client.properties file
	Required properties:

**bootstrap.servers:** Address of the Confluent Cloud broker. Can be found by going to the Confluent Cloud console, click on the Environment where the cluster resides, click on the cluster, click on ‘Cluster Settings’, and then the value is the ‘Bootstrap server’ under the ‘General’ tab.
**security.protocol:** Set this to SASL_SSL to enable encryption.
**sasl.mechanisms:** Set this to OAUTHBEARER to tell the application and Confluent to use the OAuth2.0 JWT tokens.
**sasl.oauthbearer.method:** Set this to OIDC to allow authentication with a third party application.
**sasl.oauthbearer.scope:** The entity to scope the token for. The format should be api://[client ID]/.default
**sasl.oauthbearer.token.endpoint.url:** The endpoint to generate and respond with a valid token. This value can be found by going to the Azure AD portal. Click on ‘App registrations’, then click on the application registered in step 1, click on ‘Endpoints’, and the url to use is ‘OAuth 2.0 token endpoint (v2)’.
**sasl.oauthbearer.client.id:** This is the application identity registered in Azure AD. This value can be found by going to the Azure AD portal. Click on ‘App registrations’, then click on the application registered in step 1, and then ‘Application (client) ID’. 
**sasl.oauthbearer.client.secret:** This is an application secret generated in Azure AD. To generate a secret value go to the Azure AD portal. Click on ‘App registrations’, then click on the application registered in step 1, then click on ‘Certificates & secrets’, then click on ‘+ New client secret’. The value will be the secret value generated. These secrets can be re-generated if the secret value becomes lost or expired. 
**sasl.oauthbearer.extensions:** Represents the Confluent Cloud cluster(logical cluster ID) to connect to and which identity pool to authenticate with. The format must follow: logicalCluster=[Confluent Cloud logical cluster ID], identityPoolId=[identity pool ID in Confluent Cloud]. The logical cluster ID can be found by going to the Confluent Cloud console. Click on the Environment where the cluster resides, click on the cluster, click on ‘Cluster Settings’, and then the value is the ‘Cluster ID’ under the ‘General’ tab. The identity pool ID can be found by going to the Confluent Cloud console. Click on the hamburger icon in the upper right corner, then click on ‘Accounts & access’, then click on the ‘Identity Providers’ tab, click on the IDP registered in step 2, the value is the ‘Pool ID:’ under the identity pool name.


Example values of a properties file: \
  `bootstrap.servers=pkc-abcd1.us-east-2.aws.confluent.cloud:9092` \
  `security.protocol=SASL_SSL` \
  `sasl.mechanisms=OAUTHBEARER` \
  `sasl.oauthbearer.method=OIDC` \
  `sasl.oauthbearer.scope=api://12345678-abcd-abcd-1234-1234abcd/.default` \
  `sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/0893715b-959b-4906-a185-2789e1ead045/oauth2/v2.0/token` \
  `sasl.oauthbearer.client.id=12345678-abcd-abcd-1234-1234abcd` \
  `sasl.oauthbearer.client.secret=1234abcd1234abcd1234abcd1234abcd` \
  `sasl.oauthbearer.extensions="logicalCluster=lkc-vwxyz,identityPoolId=pool-12ab"`



## Troubleshooting Tips
In the Azure AD portal, make sure the Manifest in ‘App registrations’ has the key/value accessTokenAcceptedVersion = 2. If this value is missing or has a value of 1, this will result in a v1 JWT token being generated and will not work with Confluent Cloud. 



In client config.properties file, make sure the value for sasl.oauthbearer.scope is in the format api://[client id]/.default



If this error is encountered: \
`{"error":"invalid_resource","error_description":"AADSTS500011: The resource principal named api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx was not found in the tenant named…. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You might have sent your authentication request to the wrong tenant…}` \
Add the correct application client id in Azure AD → App registrations → Manifest: \
`"identifierUris": [“api://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"]` \



If you get the error`CLAIM_ISSUER_POOL_FILTER_MISMATCH`, there’s a couple things you will need to check: \
First, retrieve a sample token. \
Run the following command replacing client_id, client_secret, and tenant_id from your config.properties: \
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=[client_id]&scope=api://[client_id]/.default&client_secret=[client_secret]&grant_type=client_credentials' https://login.microsoftonline.com/[tenant_id]/oauth2/v2.0/token

Copy the access_token from the returned payload. 

Go to https://jwt.io/ or https://token.dev/, paste your token into the Encoded text box.

Make sure the issuer url (‘iss:’) is the same as what is shown in your IDP configured in the Confluent Cloud UI. If it is not, edit your IDP configured in Confluent to match this issuer url from Azure AD. This value can be found by going to the Azure AD portal. Click on ‘App registrations’, then click on the application registered in step 1, click on ‘Endpoints’, and the url to use is ‘OAuth 2.0 token endpoint (v2)’.


Make sure the filter applied for the identity pool in Confluent Cloud UI and the config.properties will match the attributes and values in the JWT decrypted payload. An example identity pool with the filter “has(claims.sub)” would accept a token with a ‘sub’ key and value. 








