
## Adobe I/O Events - Java SDK - IMS Library

This Java library wraps http API endpoints exposed by 
[Adobe Identity Management System (IMS)](https://www.adobe.io/authentication/auth-methods.html#!AdobeDocs/adobeio-auth/master/AuthenticationOverview/AuthenticationGuide.md)

### Service Account Integration (JWT authentication flow)

A Service Account connection allows your application to call Adobe services on behalf of 
the application itself or on behalf of an enterprise organization.

For this type of connection, you will create a JSON Web Token (JWT) that encapsulates 
your credentials and begin each API session by exchanging the JWT for an access token.

The JWT encodes all of the identity and security information required to obtain an access 
token and must be signed with the private key that is associated with a public key certificate specified on your integration.

This Java library will help you implement this JWT exchange token flow, to get a valid access token
and start interacting with the many Adobe I/O API that support such authentication.

#### Test Drive

    JwtTokenBuilder jwtTokenBuilder = JwtTokenBuilder.build(); // [1]
    AccessToken accessToken = ImsServiceImpl.build(jwtTokenBuilder).getJwtExchangeAccessToken(); // [2]

* [1] we instantiate a `JwtTokenBuilder` that configures itself reading our System environment variables 
* [2] we stuff it on our IMS service wrapper, and have this service retrieve an access token using a jwt exchange token flow

Have a look at our [ImsService `main()` Test Drive](./src/test/java/com/adobe/ims/ImsServiceTestDrive.java)


#### Our reusable `OpenFeign` JWT (exchange token flow) Authentication `RequestInterceptor`

This lib also contains JWT (exchange token flow) Authentication `RequestInterceptor`: [JWTAuthInterceptor](./src/main/java/com/adobe/ims/feign/JWTAuthInterceptor.java) 
It is a [Open Feign RequestInterceptor](https://github.com/OpenFeign/feign#request-interceptors).
It can be leverage to add the authentication headers expected by many Adobe APIs, it will add
* an `Authorization` header with a `Bearer` access token (generated from a JWT exchange flow)
 * renewing it only when expired or not present yet
* a `x-api-key` header matching your JWT token

#### Configurations

First, browse our [Service Account Integration (JWT authentication flow) doc](https://www.adobe.io/authentication/auth-methods.html#!AdobeDocs/adobeio-auth/master/AuthenticationOverview/ServiceAccountIntegration.md), 
then load your `JWT authentication flow` configurations in this ims sdk, you can either: 
* use a `.properties` file, see our [sample config file](./src/test/resources/ims.properties)
* use system environment variables (containing the same entries as in the above sample config file)
* use runtime `java.util.Properties` or `Map<String, String>` inputs

##### Create and configure your public and private key

First, use openssl to create an RSA private/public certificate pair

     openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate_pub.crt

You then have 3 options to configure/use this privateKey:
* Option 1: use a pcks8 file (using the `pkcs8_file_path` config)
* Option 2: use base 64 encoded pcks8 key (using the `encoded_pkcs8` config) 
* Option 3: use a keystore (using the `pkcs12_file_path`, `pkcs12_alias` and `pkcs12_password` config)

For option 1, to convert your private key to a PKCS8 format, use the following command: 

    openssl pkcs8 -topk8 -inform PEM -outform DER -in private.key -nocrypt > private.pkcs8.key

For option 2, to base 64 encode it, use the following command: 

    base64 private.pkcs8.key 
    
For option 3, Use the following commands to set the alias (as `myalias` here)  and a non-empty keystore password.

    cat private.key certificate_pub.crt > private-key-crt
    openssl pkcs12 -export -in private-key-crt -out keystore.p12 -name myalias -noiter -nomaciter

#### your JWT token claim
 
FYI, the JWT token will generate for you will contain the following claims:
* `exp` - the expiration time. IMS allows a time skew of 30 seconds between the time specified and the IMS server time.
* `iss` - the issuer. It must be present, and it must be in the format: `org_ident@AdobeOrg` It represents the identity of the organization which issued the token, and it must be for an organization that has provided IMS with a valid certificate. 
* `sub` - the subject. It must be present and must be in the format: `user_ident@user_auth_src`. It represents the ident and authsrc of the technical account for which a certificate has been uploaded to IMS
* `aud` - the audience of the token. Must be only one entry, in the format: `ENDPOINT_URI/c/client_id`, where the client_id is the client id for which the access token will be issued. The `ENDPOINT_URI` must be a valid IMS endpoint (e.g. `https://ims-na1.adobelogin.com` for IMS production)
* `one or more metascopes claims`, in the format: `ENDPOINT_URI/s/SCOPE_CODE: true`, where the ENDPOINT_URI has the same meaning as for the audience, and the SCOPE_CODE is a valid meta-scope code that was granted to you when the certificate binding was created.

Note that Optionally, the JWT can contain the following claims (not implemented here yet)
* `jti` - a unique identifier for the token. It is dependent on the setting being configured when the certificate binding was created, and if it is set as required it must have not been previously seen by the service, or the request will be reject

It will also help you getting this signed with a `RSASSA-PKCS1-V1_5` Digital Signatures with `SHA-2` and a `RS256` The JWT algorithm/`alg` header value.
For this, it leverages a third-party open source library : [jjwt](https://github.com/jwtk/jjwt)

## Builds

This Library is build with [maven](https://maven.apache.org/) (it also runs the unit tests):

### Contributing

Contributions are welcomed! Read the [Contributing Guide](../.github/CONTRIBUTING.md) for more information.

### Licensing

This project is licensed under the Apache V2 License. See [LICENSE](../LICENSE.md) for more information.

  