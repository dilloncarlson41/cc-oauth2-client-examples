## Starting up and running your dotnet application

# Setting up Confluent Cloud

# Setting up Azure App Registrations

# Properties File Values

Below are the values you'll need to supply in the client.properties file to authenticate to Confluent Cloud via OAuth2 



Example with fake values to demonstrate format:

    bootstrap.servers=abc-1234.us-east-1.aws.confluent.cloud:9092
    security.protocol=SASL_SSL
    sasl.mechanisms=OAUTHBEARER
    sasl.oauthbearer.method=OIDC
    sasl.oauthbearer.scope=api://881234-1e2e-3e4e-5e6e-885tz345/.default
    sasl.oauthbearer.token.endpoint.url=https://login.microsoftonline.com/012345b-123b-1234-1234-1234e1234/oauth2/v2.0/token
    sasl.oauthbearer.client.id=881234-1e2e-3e4e-5e6e-885tz345
    sasl.oauthbearer.client.secret=asdf1234asdf1234asdf1234
    sasl.oauthbearer.extensions="logicalCluster=lkc-asdf,identityPoolId=pool-asdf"
    ssl.ca.certificate.stores=CA,Root

# Commands to build and run the producer:

    cd \dotnet-client\producer
    dotnet build producer.csproj
    dotnet run $(pwd)/../client.properties                                                         