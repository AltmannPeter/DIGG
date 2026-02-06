# An EUDIW Attestation

This text details an EUDIW PID attestation in one of the two core formats, SD-JWT VC, and the technical requirements for its issuance using OpenID for Verifiable Credential Issuance (OID4VCI).

## Attestation Requirements

### An SD-JWT VC PID

```json
{
   "iss":"https://pid-issuer.aendgard.example",
   "vct":"urn:eudi:pid:aendgard:1",
   "given_name":"Astrid",
   "family_name":"Holmgren",
   "birthdate":"1978-04-10",
   "place_of_birth":{
      "locality":"Ændholm",
      "country":"Kingdom of Ændgard"
   },
   "address":{
      "street_address":"Sjøgata 12",
      "locality":"Viken",
      "postal_code":"12001",
      "country":"Kingdom of Ændgard"
   },
   "nationalities":[
      "Ændgard"
   ],
   "gender":"male"
}
```

The SD-JWT payload:

```json
{
  "_sd": [
    "2r009dzvHuVrWrRXT5kJMmHnqEHHnWe0MLVZw8PATB8",
    "3NdDJYRwBqaEIyXCHcn4a5YKJagPgIP0Wkg9AorgAKg",
    "5fn6IiXjUPmGQ4mYHxDdt6EjvBQ7gRvGcdEnSCEN8Ek",
    "6BzMBk7FGivogbldO75f3NLwjFliCqb7nn1GhgCokrQ",
    "90CT8AaBPbn5X8nRXkesju1i0BqhWqZ3wqD4jF-qDGk",
    "9nEwUrdeZ6kcRbEj52QH3nOR-oie0_p9tMNIAEuFvMU",
    "HTh6Zr2J8aiqpa963cLkuKeQDf9O00FzOHhyjGpVfTg",
    "JCp2K2NhH-LE5L0j7ME0LNx-T-px9QFtamSsxmCUCxU",
    "LoMGrhlDm_bUWUX1gdhRxTsy4d4zywGzlSVocRSY01M",
    "UgUjq5whzQewO59fVNj0f0mqyV8YPXU54sp4kLecwQk",
    "Y0H-E3hzKEUnAyri_x9tT9k-j4i65ao9hkeEHNexosI",
    "ezCAmaTwI2nPj_1IiB3KZfiEORLeRwhZsMMDMZIJ8kA",
    "i0H_-WAHwfEjt8tqQH74uOCWvquY3FwuX-kx4e2RJH8",
    "r1GrH46yDV62sezImEjxa6cJSZMT212jrOSHiec9ytM",
    "rVj3RLs7K8zdNy7zXz1Cr39Y6rog8ulNozk4F7NCuC0",
    "t5kp0HfxMS1eHNCGRzrK9Gi5oTZreg5Ue9lLQIuLvB4"
  ],
  "iss": "https://pid-issuer.aendgard.example",
  "iat": 1683000000,
  "exp": 1883000000,
  "vct": "urn:eudi:pid:aendgard:1",
  "_sd_alg": "sha-256",
  "cnf": {
    "jwk": {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  }
}
```

Where possible, IANA claims MUST be followed. Additional ones to explain are:

- `vct`: A string that acts as a unique identifier or schema for the credential. It defines the semantics of the respective claims and associated rules.
- `_sd`: An array of digests that represent individual attributes hidden until selectively disclosed by the user.
- `_sd_alg`: The cryptographic hash algorithm (e.g., SHA-256) used to compute the attribute digests.

### Catalogues and Rulebooks

Each attribute mentioned in Annex VI to Regulation (EU) No 910/2014 must be registered with the EC. Optional attributes do not need to be registered. The purpose of the registration is to enable QTSPs issuing QEAAs to verify the attributes against authentic sources. The catalogue provides the information necessary to locate the verification points at the national level for a specific attribute. Each attribute registration must also include a semantic description of the attribute, namespace identifiers, data types etc. [Article 7 in CIR 2025/1569](https://eur-lex.europa.eu/eli/reg_impl/2025/1569/oj/eng) contains more detail.

Each attestation type also needs to register its entire schema to the EC as per [Article 8 in CIR 2025/1569](https://eur-lex.europa.eu/eli/reg_impl/2025/1569/oj/eng).

Related to the CIR 2025/1569 requirements is the concept of the "rulebook." The ARF defines rulebooks as a way to provide details important for schemas and attributes. It is the human-readable authoritative documentation that explains what a specific attestation represents (e.g., a diploma, a driving licence) and how it works, detailing identifiers, semantics, encodings, and trust models. So in addition to a schema and attribute registry, you also need two complementary artefacts to define an attestation type:

1. The attestation rulebook (human readable)
2. The attestation schema (machine validation)

Further details on how the rulebooks and catalogues relate is available in [TS 11](https://raw.githubusercontent.com/eu-digital-identity-wallet/eudi-doc-standards-and-technical-specifications/refs/heads/main/docs/technical-specifications/ts11-interfaces-and-formats-for-catalogue-of-attributes-and-catalogue-of-schemes.md).

In sum:

- The Catalogue is the directory where users find available attestations.
- The Rulebook is the manual (documentation) describing the rules of a specific attestation.
- The Scheme (machine-readable) is the code/specification software uses to process the attestation

### IT Planning for Attestation Support

1. Semantic Mapping and Data Governance: Internal data must be translated into the exact structures required by the European Commission. This entails:
   - Schema: Map internal fields to the precise claim paths defined in the Attestation Schema
   - Attribute Registry: Ensure all mandatory attributes required by Annex VI of Regulation (EU) No 910/2014 are retrieved from "Authentic Sources".
   - Vocabulary Definition: For private claims, define and host documentation (the Rulebook) explaining the semantics and data types so Verifiers can interpret the information correctly.
2. Support for Selective Disclosure: Identify which claims should be individually disclosable. Those that are disclosable need then to be prepared as per the IETF SD-JWT specification.
3. Lifecycle Management & Revocation: An attestation is a living document that must accurately reflect the current status of the holder. This means:
   - If supporting long-lived tokens: develop automated triggers that update the Token Status List (TSL) as soon as an internal status changes (e.g., a professional license is revoked or an ID is reported stolen).
   - Manage the private keys used for signing specific attestation types within a Hardware Security Module (HSM), ensuring they are distinct from keys used for other corporate functions.
   - Update catalogues and rulebooks when required.

To support high-assurance issuance, an issuer must support the OID4VCI HAIP profile. Beyond the OAuth 2.0 support required for HAIP, there is a need to setup the attestation service endpoint that the user can use to request an attestation. This include data mapping, session logic, signing logic etc. The issuer also needs to authenticate the user to know that the issued attestation is bound to the identity subject it describes. The specific authentication process depends on whether this is done with an EUDIW PID, or using another means. Using other means to establish the user identity is out of scope for this text. If the user authenticates with a PID, the issuer needs to validate both the EUDIW using a WUA and a WIA (essentially a key attestation and a software attestation).

The issuer also needs to do metadata management. Hosting the `.well-known/openid-credential-issuer` endpoint to declare technical capabilities and display information.
