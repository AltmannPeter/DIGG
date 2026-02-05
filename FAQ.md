Many topics and questions are already covered at a high level on the EC sites for [Issuers](https://ec.europa.eu/digital-building-blocks/sites/spaces/EUDIGITALIDENTITYWALLET/pages/881984686/Wallet+for+Issuers) and for [Service Providers](https://ec.europa.eu/digital-building-blocks/sites/spaces/EUDIGITALIDENTITYWALLET/pages/881984674/Wallet+for+service+providers). These two pages are great for giving people a general idea of what the Wallet will be and how it will function.

With the above in mind, let us look at specific questions and comments. In general, I do not believe we should keep creating copies and flavors of information that already exists and that people can freely and readily access on their own. So where possible, I will reply by referencing material to read.

> Jag pratade med Lotta senare under dagen igår och vi kom tillbaka till behovet av att kunna förklara hur attributintyg utfärdas till och presenteras från plånboken. Det inkluderar hur den inledande autentiseringen utförs.

The two above links already explain this at a high level. For additional details, the ARF section [6.6.2](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/architecture-and-reference-framework-main.md#662-pid-or-attestation-issuance) is dedicated to the topic of issuance. Additional details on issuance can be found in [Annex 2 section 2.3.7](https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/main/docs/annexes/annex-2/annex-2.02-high-level-requirements-by-topic.md#a237-topic-10---issuing-a-pid-or-attestation-to-a-wallet-unit) and the associated legal text is found in [CIR 2024/2977](https://eur-lex.europa.eu/eli/reg_impl/2024/2977/oj). Protocol details for both issuance and presentations are available in the [HAIP](https://openid.net/specs/openid4vc-high-assurance-interoperability-profile-1_0-ID1.html#name-openid-for-verifiable-crede) and in the therein linked specifications.

How the initial authentication is done depends on what you mean.

1. First authentication during onboarding: Only the Wallet Provider does this so it does not impact anyone else. Requirements are listed in the ARF (and are sane).
2. Authentication during issuance: Covered in the above links. The core idea is that the user will use a PID and selectively disclose the attributes required for authenticating to the RP. How exactly this process unfolds, depends entirely on how the flow is initiated. It can be initiated user side or RP side as per OID4VP.

> Du var inne på att intressenterna för varje typ av attributintyg måste komma överens om innebörd och utformning. [...] Det är därför inte [överenskommelserna] vi behöver förklara utan hur det går till när man ska omsätta överenskommelserna i attributscheman mm, utfärda attributintygen till plånboken och sedan kommunicera med en verifier som tar emot och kontrollerar attributintygen.

How to select attribute names depends on the approach you pick. There are two main approaches supported by the Wallet:

1. (REQ) non-linked data: This is used by SD-JWT VC and mdoc-MSO. Here, agreements are declared elsewhere and both issuers and verifiers need to know where that agreement exists and is published. Some agreements (e.g., the IANA ones) are publicly available and everyone knows where to find them. Others may need to be registered and made public that way. A third option is to have private attributes that are not registered. In that case, it is up to the issuer and verifier to figure out how to agree on what those attributes mean. This can be done using e.g., rulebooks.
2. (OPT) linked data: This is used by JSON-LD. Agreements are declared and published at an uri and the attestation contains pointers to these uris.

So specifically, the way you translate an agreement into attribute names to include in the attestation depends on the format you use. The required formats do not use linked data, meaning that you need to know where to find the associated namespace / definitions. Let us look at an example mdoc with an birthdate, issuance date, and expiration information:

```json
{
  "id": "8669f836-b3b2-43f3-aecd-d92187cb1cac",
  "encoded": "omp...QVU=",
  "decoded": {
    "namespaces": {
      "org.iso.18013.5.1": [
        {
          "digestID": 4,
          "random": "5levHsGtTQiVO30tX/fSmg==",
          "elementIdentifier": "birth_date",
          "elementValue": {
            "type": "date",
            "value": "1990-04-30"
          }
        },
        {
          "digestID": 3,
          "random": "5wACvuuwC6fo3TXf/fb3bQ==",
          "elementIdentifier": "issue_date",
          "elementValue": {
            "type": "date",
            "value": "2024-01-01"
          }
        },
        {
          "digestID": 13,
          "random": "+UqiYYLdKafjOiK3u7CcGw==",
          "elementIdentifier": "expiry_date",
          "elementValue": {
            "type": "date",
            "value": "2035-12-30"
          }
        }
      ]
    },
    "issuerAuth": "hEO...UkA=="
  }
}
```

Attribute claims are structured in namespaces. Each attribute under `namespaces` is an object containing other attributes. If an attribute is found in a namespace, the interpretation of that attribute is done using the namespace it is under. In the above example, all attributes need to be interpreted as defined in `org.iso.18013.5.1`, i.e., ISO 18013-5. For a full example of how this is done, see [this MATTR guide](https://learn.mattr.global/docs/concepts/mdocs/structure-to-function).

Correspondingly, you have options also in SD-JWT VC. But here, there are multiple ways to communicate what agreement is used. One way is to use the credential type value. The issuer metadata includes information about formats and types the user can request. Each type is associated with a declaration of what each attribute value means.

In both mdoc and SD-JWT, when issued and requested over OID4VC you have ways to declare the `doctype` or `vct`, i.e. what agreement is to be used. For issuance, see [OID4VCI](https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html). Presentation is format specific and details are found in the [OID4VP spec](https://openid.net/specs/openid-4-verifiable-presentations-1_0.html).
