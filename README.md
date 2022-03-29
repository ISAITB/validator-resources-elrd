# ELRD Validator

This repository contains the configuration for the **ELRD validator**. This is available as a standalone validator and also as the validation service used as part of ELRD conformance tests.

The ELRD validator is an instance of the [XML validator service](https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html) of the [Interoperability Test Bed](https://joinup.ec.europa.eu/collection/interoperability-test-bed-repository/solution/interoperability-test-bed).

## Configuration

The files you may edit to adapt the validation are as follows:
* Folder `resources\v*.*` contains the files for the different ELRD versions.
* File `resources\config.properties` contains the validator's configuration that you may want to adapt.

The key resource here is the **config.properties** configuration file that defines the validator's core setup. The configuration properties used in this file are documented [here](https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html#validator-configuration-properties). 

To publish changes commit and push your updates. In 1-2 minutes the online service will be updated.

## Usage

The online service is used via:
* User interface at https://www.itb.ec.europa.eu/elrd/upload (instructions [here](https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html#validation-via-user-interface)).
* Web service API at https://www.itb.ec.europa.eu/elrd/api/validation?wsdl (instructions [here](https://www.itb.ec.europa.eu/docs/guides/latest/validatingXML/index.html#validation-via-soap-web-service-api)).

Specifically on the validator's SOAP API for machine-to-machine integrations, you would use it by making an HTTP `POST` to https://www.itb.ec.europa.eu/elrd/api/validation with the following SOAP payload:

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:v1="http://www.gitb.com/vs/v1/" xmlns:v11="http://www.gitb.com/core/v1/">
   <soapenv:Header/>
   <soapenv:Body>
      <v1:ValidateRequest>
        <!-- Sample to validate an instance provided as-is. -->
         <input name="xml" embeddingMethod="STRING">
            <v11:value><![CDATA[<foo><bar>data</bar></foo>]]></v11:value>
         </input>
        <!-- Sample to validate a published instance.
         <input name="xml" embeddingMethod="URI">
            <v11:value>https://path.to/elrd_content.xml</v11:value>
         </input>
         -->
         <input name="type" embeddingMethod="STRING">
            <v11:value>latest</v11:value>
         </input>
         <input name="locationAsPath" embeddingMethod="STRING">
            <v11:value>true</v11:value>
         </input>
         <input name="addInputToReport" embeddingMethod="STRING">
            <v11:value>false</v11:value>
         </input>
      </v1:ValidateRequest>
   </soapenv:Body>
</soapenv:Envelope>
```

## Deployment via Docker

Running an on-premise instance (or instances) of the ELRD validator is documented in the [validator production installation guide](https://www.itb.ec.europa.eu/docs/guides/latest/installingValidatorProduction/index.html). You have [several options](https://www.itb.ec.europa.eu/docs/guides/latest/installingValidatorProduction/index.html#step-2-install-the-validator) to run a local instance with the simplest being to define and use your own [Docker image](https://www.itb.ec.europa.eu/docs/guides/latest/installingValidatorProduction/index.html#step-2-install-the-validator).

In brief you need to do the following steps:

1. Clone the current repository in e.g. folder `/validator`.
2. Define within `/validator` a `Dockerfile` with the following contents:

```
FROM isaitb/xml-validator:latest
COPY resources /validator/resources/
ENV validator.resourceRoot /validator/resources/
ENV validator.domainName.resources elrd
```

3. Build the validator using `docker build -t imola/elrd-validator .`
4. Run the validator using `docker run -d --name elrd-validator -p 8080:8080 imola/elrd-validator:latest`

Completing the above steps, the validator will be available via:
* User interface at http://localhost:8080/elrd/upload.
* Web service API at http://localhost/elrd/api/validation?wsdl.

As an alternative to using docker build and run, you may also use Docker-compose. To do so, define in `/validator` file `docker-compose.yml` with the following contents:

```
version: '2.1'
services:
   elrd-validator:
      image: isaitb/xml-validator:latest
      restart: unless-stopped
      ports:
       - "8080:8080"
      volumes:
       - ./resources/:/validator/resources/
      environment:
       - validator.resourceRoot=/validator/resources/
       - validator.domainName.resources=elrd

```

From this point you can build and run the validator using `docker-compose up -d`.