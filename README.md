# blend-template-connector-api
This is a project skeleton that developers can use as a starting point when creating API Implementations with Anypoint Studio. This skeleton should help drive consistent use of best practices within the team. It is configured to support CiCd, Splunk logging  Mulesoft Integrations Platform (CURRENT)at Blend

# Modifications needed after cloning the template:
1. The keystore/truststore for the blend-template-connector-api is only for reference. You will have to create new keystore/truststore after you setup your api because the certs depend on the API name that's part of the URI.
2. Update the service name in /src/main/resources/log4j2.xml
3. Rename references to the template in the POM file with your own service/API name
