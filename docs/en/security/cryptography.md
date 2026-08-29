# Cryptography Applied to MuleSoft

[Versão em português](../../pt/seguranca/criptografia.md)

Fundamentals, certificates, TLS, and the Cryptography Module.

## 1. Cryptography Fundamentals

Cryptography transforms readable data (plaintext) into encrypted data. Only someone with the appropriate key should be able to recover the original content. Historical and modern techniques include:

**Substitution:** replaces each symbol in the original text with another according to a rule. The Caesar cipher is a classic example.

**Transposition:** preserves the symbols but changes their positions according to a rule.

**Steganography:** conceals the existence of a message, for example by embedding data in images, audio, or video. It does not replace cryptography; both techniques can be combined. To an observer, the file may appear ordinary even though it contains hidden data.

**Product ciphers:** perform successive rounds that combine substitution and transposition. AES applies this principle in a complex way at the bit level.

### 1.1 Hashing and encryption: fundamental differences

| Characteristic | Encryption                                       | Hashing                                                   |
| -------------- | ------------------------------------------------ | --------------------------------------------------------- |
| Reversible?    | Yes — the correct key recovers the original data | No — it is a one-way function                             |
| Goal           | Confidentiality                                  | Integrity and verification                                |
| Output size    | Varies with the input                            | Fixed; SHA-256 always produces 256 bits                   |
| Uses a key?    | Yes, symmetric or asymmetric                     | No; password hashing may use a _salt_, which is not a key |
| Example        | Encrypting a file                                | Storing passwords or verifying files                      |

As a practical rule, use encryption when the original data must be recovered. Use hashing when you only need to verify that data has not changed or compare values without storing the original, as with passwords.

Hashing is not a weak form of encryption. A hash is not "decrypted"; an attacker may instead try to discover its input through brute force or precomputed tables (_rainbow tables_).

### 1.2 Hash algorithms

- **MD5:** produces 128 bits and has practical collisions. It must not be used for security controls; it may appear only as a non-adversarial checksum.
- **SHA-1:** produces 160 bits and has practical collisions. It should be avoided in new projects.
- **SHA-2:** a family that includes SHA-256 and SHA-512, widely used for integrity checks and digital signatures.
- **SHA-3:** has a different internal structure (Keccak) and is an alternative to SHA-2.

For passwords, a plain hash, even SHA-256, is not recommended because it is too fast and enables large-scale brute-force attacks. Password hashing therefore uses deliberately slow functions such as:

- bcrypt;
- scrypt;
- Argon2id, configured with parameters appropriate for the environment.

### 1.3 Symmetric encryption

The same key encrypts and decrypts the data. It is fast and suitable for large volumes. The challenge is sharing the secret key securely.

- **AES:** a widely adopted standard with 128-, 192-, or 256-bit keys. It should be used in an authenticated mode such as GCM whenever possible.
- **DES:** an old algorithm with a 56-bit key that is now insecure.
- **3DES:** applies DES three times. It is more secure than DES but slow and being phased out.
- **Blowfish and Twofish:** alternatives to AES that are less commonly used today.

### 1.4 Asymmetric encryption

Uses a related key pair: a public key that may be shared and a private key that must remain secret. It is generally used to establish keys, encrypt small values, or create digital signatures. Large volumes of data are protected with hybrid encryption.

- **RSA:** based on the difficulty of factoring large numbers. Used in TLS, PGP, and digital signatures.
- **ECC:** provides security with smaller and more efficient keys. It is used, for example, in modern SSH keys.
- **Diffie-Hellman (DH/ECDH):** allows two parties to establish a shared secret over an insecure channel.
- **DSA/ECDSA:** used for digital signatures, not encryption.

In modern HTTPS, public-key mechanisms authenticate the parties and help establish session secrets. The traffic itself uses authenticated symmetric encryption.

## 2. Digital Certificates

A digital certificate binds a public key to an identity such as a person, company, or server. It is signed by a certificate authority (CA) or, in PGP, may participate in a _web of trust_. It contains:

- the owner's public key;
- identity information;
- a validity period;
- the issuer's signature.

The certificate does not contain the private key. The private key is stored separately and is usually password-protected.

### 2.1 Certificate and key formats

#### Encodings

Encoding defines **how the same cryptographic data is represented** in a file. It does not change the key or certificate, nor does it increase security. For example, an X.509 certificate may be represented as Base64 text in PEM or as binary bytes in DER.

| Format | Characteristics                                                                                                       |
| ------ | --------------------------------------------------------------------------------------------------------------------- |
| PEM    | Base64 text with headers such as `-----BEGIN CERTIFICATE-----`. Extensions include `.pem`, `.crt`, `.cer`, and `.key` |
| DER    | Binary representation. Extensions include `.der` and `.cer`                                                           |

PEM and DER can represent the same information; only the text or binary encoding changes. The tool that consumes the certificate must support the selected encoding.

#### Containers

A container defines **which cryptographic objects are grouped in a file**. Depending on the format, it may hold a certificate, a private key, a certificate chain, or multiple entries. Some containers, such as PKCS#12 and JKS, may also be password-protected.

| Format                       | What it stores                                               | Typical use                       |
| ---------------------------- | ------------------------------------------------------------ | --------------------------------- |
| CRT/CER                      | Public certificate; sometimes a chain                        | Distribution for validation       |
| KEY                          | Private key                                                  | Kept on the server                |
| PKCS#12/PFX (`.p12`, `.pfx`) | Certificate, private key, and chain, protected by a password | Import and export between systems |
| JKS (`.jks`)                 | Java container for certificates and keys                     | JVM applications, including Mule  |
| PKCS#7 (`.p7b`, `.p7c`)      | Certificates or chains without private keys                  | Certificate-chain distribution    |

JKS and PKCS#12 are common in the Mule and Java ecosystem because Mule Runtime runs on the JVM.

### 2.2 Keystores and truststores in Mule

The distinction is conceptual rather than a file-format distinction; both may use `.jks` or `.p12`:

- **Keystore:** stores your private keys and certificates. It is used to prove your identity.
- **Truststore:** stores public certificates from trusted entities such as CAs and partners.

### 2.3 Keystores and truststores in the TLS handshake

**Server (keystore):**

- stores its certificate and private key;
- sends the public certificate to the client;
- uses the private key to prove that it owns the certificate.

**Client (truststore):**

- stores trusted certificates;
- verifies whether the received certificate was signed by a trusted CA or by a chain that ends at one;
- establishes the connection when validation succeeds.

> This describes one-way TLS. With mTLS, the client also has a keystore and the server has a truststore. Both parties authenticate each other, which is common in API-to-API integrations in Mule.

In Mule, KBE and XML operations typically use X.509 certificates in JKS, PKCS#12, PEM, or DER. PGP uses PGP keyrings rather than X.509 certificates.

### 2.4 Self-signed certificates with keytool

`keytool` is a command-line utility distributed with the JDK. It can generate key pairs, create certificates, import and export certificates, and manage keystores. It is particularly useful in the MuleSoft ecosystem because Mule Runtime runs on the JVM and uses Java-compatible keystore formats.

Prerequisites: Java installed, `JAVA_HOME` configured, and `keytool` available on the `PATH`.

> Self-signed certificates are suitable for testing and controlled environments. For public production endpoints, prefer certificates issued by a trusted CA.

#### 1. Generate a `.p12` keystore

```shell
keytool -genkeypair -alias my-api-keystore -keyalg RSA -keysize 2048 -validity 365 -keystore my-api-keystore.p12 -storetype PKCS12
```

| Parameter           | Purpose                              |
| ------------------- | ------------------------------------ |
| `-genkeypair`       | Generates the key pair               |
| `-alias`            | Identifies the entry in the keystore |
| `-keyalg RSA`       | Selects the algorithm                |
| `-keysize 2048`     | Sets the key size                    |
| `-validity 365`     | Sets the validity in days            |
| `-keystore`         | Sets the output file                 |
| `-storetype PKCS12` | Sets the container format            |

Export the public certificate:

```shell
keytool -exportcert -alias my-api-keystore -keystore my-api-keystore.p12 -storetype PKCS12 -file my-public-cert.crt
```

#### 2. Generate a `.pfx` keystore

`.pfx` and `.p12` represent the same PKCS#12 format. Only the file name changes:

```shell
keytool -genkeypair -alias my-api-keystore -keyalg RSA -keysize 2048 -validity 365 -keystore my-api-keystore.pfx -storetype PKCS12
keytool -exportcert -alias my-api-keystore -keystore my-api-keystore.pfx -storetype PKCS12 -file my-public-pfxcert.crt
```

The keystore remains on the server, is password-protected, and is never shared. The exported `.crt` goes into the client's truststore.

#### 3. Import the certificate into the truststore

```shell
keytool -importcert -alias trusted-demo-api -file my-public-cert.crt -keystore client-truststore.p12 -storetype PKCS12
```

| Parameter           | Purpose                          |
| ------------------- | -------------------------------- |
| `-importcert`       | Imports a certificate            |
| `-alias`            | Identifies the truststore entry  |
| `-file`             | Specifies the public certificate |
| `-keystore`         | Specifies the destination file   |
| `-storetype PKCS12` | Sets the format                  |

If the truststore does not exist, `keytool` creates it and asks for a password. For a self-signed certificate, it may also ask you to confirm that the certificate should be trusted.

```text
SERVER                                        CLIENT
------                                        ------
1. genkeypair → my-api-keystore.p12
   (private key + certificate)

2. exportcert → my-public-cert.crt
                                      ──► 3. importcert → client-truststore.p12
```

The server has a keystore containing its identity and private key; the client has a truststore containing the trusted public certificate. This concept is used by the Mule HTTP Listener and HTTP Requester.

## 3. TLS in the MuleSoft HTTP Listener

![TLS and keystore configuration in the HTTP Listener](../../assets/images/criptografia/tls-listener.png)

### 3.1 CloudHub 2.0 and Last-Mile Security

In CloudHub 2.0, the certificate presented at the edge is independent of the certificate configured in the HTTP Listener. The request passes through two layers.

#### External layer: edge and ingress

- The public URL first reaches the Ingress Controller.
- The presented certificate is MuleSoft's default certificate for `*.cloudhub.io` or the certificate for a configured custom domain.
- The first TLS connection is established at this layer.

#### Internal layer: Last-Mile

![Last-Mile Security option in Runtime Manager](../../assets/images/criptografia/cloudhub-last-mile.png)

- The Ingress Controller forwards the request to the Mule application container.
- Without Last-Mile Security, this segment uses HTTP.
- With Last-Mile Security, the Ingress establishes a new internal HTTPS connection.
- The HTTP Listener certificate participates in this second connection.

A self-signed certificate may be appropriate for this segment when trust is managed within the environment. Confirm the current Private Space requirements.

**When to consider leaving it disabled:** very low latency is a priority, the infrastructure trust model is accepted, and the data is not subject to strict encryption-in-transit requirements.

**When to enable it:** internal, contractual, or regulatory requirements apply, such as PCI DSS environments, sensitive data, and _Zero Trust_ architectures.

## 4. MuleSoft Cryptography Module

The [Cryptography Module](https://docs.mulesoft.com/cryptography-module/latest/) implements **application-level cryptography**. Unlike TLS, which protects data while it travels over the network, this module enables the flow itself to encrypt, decrypt, sign, or verify the content processed by the application.

For example, a flow can encrypt a payload before saving it to a database and decrypt it only when it needs to process it again. The same concept can be applied before sending data to a queue, file, or external system. The information can therefore remain protected even after leaving the TLS channel.

1. **JCE:** provides PBE, which derives a key from a password, and KBE, which uses a supplied cryptographic key.
2. **PGP:** uses public and private keys and is common for secure file exchange between partners.
3. **XML operations:** encrypt portions of XML and apply or verify XML digital signatures.

| Strategy | Key source              | Type                    | Typical use                           |
| -------- | ----------------------- | ----------------------- | ------------------------------------- |
| JCE/PBE  | Password or passphrase  | Symmetric               | Shared password                       |
| JCE/KBE  | Directly supplied key   | Symmetric or asymmetric | Managed keys or keystores             |
| PGP      | PGP key pair            | Asymmetric/hybrid       | Secure exchange between partners      |
| XML      | Depends on the strategy | Symmetric or asymmetric | Encrypting or signing portions of XML |

### 4.1 JCE and KBE example

> **Warning:** to keep this example self-contained and educational, the password
> `Senha@123` is stored directly in the XML. In production, use Secure
> Configuration Properties or another secure secret-management mechanism. RSA
> should protect only small values or session keys. For larger payloads, use
> AES-GCM and protect the AES key with asymmetric encryption.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<mule xmlns:crypto="http://www.mulesoft.org/schema/mule/crypto" xmlns:tls="http://www.mulesoft.org/schema/mule/tls"
	xmlns:ee="http://www.mulesoft.org/schema/mule/ee/core"
	xmlns:http="http://www.mulesoft.org/schema/mule/http" xmlns="http://www.mulesoft.org/schema/mule/core" xmlns:doc="http://www.mulesoft.org/schema/mule/documentation" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd
http://www.mulesoft.org/schema/mule/ee/core http://www.mulesoft.org/schema/mule/ee/core/current/mule-ee.xsd
http://www.mulesoft.org/schema/mule/tls http://www.mulesoft.org/schema/mule/tls/current/mule-tls.xsd
http://www.mulesoft.org/schema/mule/crypto http://www.mulesoft.org/schema/mule/crypto/current/mule-crypto.xsd">
	<http:listener-config name="HTTP_Listener_config" doc:name="HTTP Listener config" doc:id="33fad091-7d83-4b8e-a40f-811c6cabbe3e" >
		<http:listener-connection host="0.0.0.0" port="${api.port}" protocol="HTTPS">
			<tls:context >
				<tls:key-store type="pkcs12" path="certs/my-api-keystore.p12" alias="my-api-keystore" keyPassword="teste@123" password="teste@123" />
			</tls:context>
		</http:listener-connection>
	</http:listener-config>
	<global-property doc:name="Global Property" doc:id="3af19b8a-ec5d-465b-a894-7f5cec71ac0b" name="env" value="dev" />
	<configuration-properties doc:name="Configuration properties" doc:id="4b14a088-78c7-4391-9143-49e3351be5b0" file="config/${env}-configuration.yaml" />
	<crypto:jce-config name="Crypto_Jce" type="PKCS12" doc:name="Crypto Jce" doc:id="bd5dde2a-6afa-4bce-99a9-f47e9840fe92" keystore="certs/my-api-keystore.p12" password="teste@123" >
		<crypto:jce-key-infos >
			<crypto:jce-asymmetric-key-info keyId="minhaChaveApi" alias="my-api-keystore" password="Teste@123" />
		</crypto:jce-key-infos>
	</crypto:jce-config>
	<flow name="testeFlow" doc:id="e2ffbc66-912c-416d-840b-ff6c6274db90" >
		<http:listener doc:name="Listener" doc:id="ac2c7fea-6048-4d20-9f55-9f5e3d9d6879" config-ref="HTTP_Listener_config" path="/p1"/>
		<ee:transform doc:name="Transform Message" doc:id="ebc7de77-39d1-484b-92ce-93cbf0959545" >
			<ee:message >
				<ee:set-payload ><![CDATA[%dw 2.0
output application/json
---
payload]]></ee:set-payload>
			</ee:message>
		</ee:transform>
		<logger level="INFO" doc:name="Logger" doc:id="9fc9a0b4-c86f-45f3-b574-110f79534ba1" message="#['\n'] #[payload] #['\n']"/>
		<crypto:jce-encrypt doc:name="Jce encrypt" doc:id="bcef53f7-df4c-44ac-a4fc-6df4557200ed" config-ref="Crypto_Jce" algorithm="RSA" keyId="minhaChaveApi"/>
		<ee:transform>
    <ee:message>
        <ee:set-payload><![CDATA[%dw 2.0
output application/json
import toBase64 from dw::core::Binaries
---
{
    encrypted: toBase64(payload)
}]]></ee:set-payload>
    </ee:message>
</ee:transform>
		<logger level="INFO" doc:name="Logger" doc:id="e24ffbbb-cae2-4790-8171-d4327013f3df" message="#['\n'] #[payload] #['\n']" />
	</flow>
</mule>

```

## 5. References

- [MuleSoft — Cryptography Module Reference](https://docs.mulesoft.com/cryptography-module/latest/cryptography-module-reference)
- [MuleSoft — CloudHub 2.0 Networking Architecture](https://docs.mulesoft.com/cloudhub-2/ch2-networking-guide)
- [MuleSoft — Configuring XML Cryptography](https://docs.mulesoft.com/cryptography-module/2.0/cryptography-module-xml)
