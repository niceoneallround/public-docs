# Trust Criteria and Trust Credentials

Trust Criteria is an abstract Trust Model term for any business, compliance, or privacy policy.

Trust Credential is an abstract Trust Model term for any signed document making a claim about an entity.

Trust Criteria and Trust Credentials are used to both represent and enforce both policies and provenance. They are cryptographically bound into a Privacy Graph by a Privacy Agent so that they are carried with the data. The Trust Criteria and Trust Credentials can be of any format or schema and may be included by value or reference.

In this document Data Provenance is a general term that covers the data's Provenance, Lineage, and Descriptive information.

# PN Trust Criteria and PN Trust Credentials

PN Trust Criteria and PN Trust Credentials are concrete types of Trust Criteria and Trust Credentials that can be used by parties and the WebShield Software to both represent and enforce both policies and provenance. Note parties can also associate other Trust Criteria and Trust Credentials with the data.

PN Trust Credentials are digitally signed documents that allow parties to assign credentials/attributes (typically globally unique URLs) from a PN Trust Model to PN Resources such as Privacy Domains and PN Data Models. A PN Trust Credential may be a JWT and/or X509 certificate. The attributes are used to enforce policies ('authorized-client'), record provenance ('secure-source'), describe data ('PII-attribute'), etc. WebShield provides a command line tool to issue/capture PN Trust Credentials.

PN Trust Criteria describe a set of required PN Trust Credentials in terms of attribute and issuers and evaluates to true for a PN Resource if that PN Resource has a super-set of the required PN Trust Credentials. PN Trust Criteria have a JSON schema defined by the WebShield software.

PN Trust Credentials and PN Trust Criteria can be used by any party and are visible in the Privacy Graph.

An example of WebShield software using PN Trust Criteria and PN Trust Credentials is the Privacy Pipe Information Flow Control mechanisms, specifically
 - the data's PN Trust Criteria that describe what PN Trust Credentials a destination Privacy Domain must have, i.e it must have a superset of the required.
 - the destination Privacy Domain's PN Trust Criteria that describe what PN Trust Credentials the data must have, i.e. it must have a superset of the required.

Some non-normative PN Trust Criteria and PN Trust Credentials

```json
//
// PN Trust Criteria statement. All the PN trust criteria need to met (AND clause)
// the scope defines that should be applied to the whole graph.
//
{ "@context": "pn jsonld context",
  "id": "<globally unique id for the trust criteria",
  "type": "pn_trust_criteria_statement",
  "scope": "graph",
  "trust_criteria":[
    {
      "type": "pn_trust_criteria",
      "attribute": "<require attribute>",
      "issuer": "<issuer of attribute>",
    },
  ],
}

//
// example PN Trust Credential JWT
//
{ "header":{},
  "payload": {
    "pn_jwtid": "<globally unqiue id>",
    "pn_jwt_type": "pn_trust_credential",
    "sub": "<subject privacy domain>",
    "iss": "<issuer privacy domain>",
    "provenance_attribute_such_as_secure_source": "1",
  },
  "signature":{},
}

```

## Associating Trust Criteria with the Data.
Both PN Trust Criteria and other Trust Criteria can be cryptographically bound to the data within a Privacy Graph. The data owner can associate Trust Criteria with data (1) in the data plane using the privacy_pipe_request see below, and (2) in the management plane using the PN Data Model see below.

The data owner can *associate PN Trust Criteria with the data to control what Privacy Domains the data can be forwarded to* as shown below

```json

//
// The optional Privacy_Pipe_Request has a destination_privacy_domain_trust_criteria
// property that can be set to the necessary PN Trust Criteria.
//
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria object",
  "other_props_removed_for_clarity": "",
}

// The PN Data model allows a party to define a default so that if none are passed
// to the privacy pipe this value is used
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria>",
  "other_props_removed_for_clarity": "",
}

//
// The PN Privacy Graph payload contains the destination_privacy_domain_trust_criteria.
// Note the PG header and digital signature are removed for clarity.
//
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "destination_privacy_domain_trust_criteria": "<pn_trust_criteria_statement>",
    "other_props_removed_for_clairty": "",
  },
  "data": []
}

```

The data owner can optionally *associate any Trust Criteria with the data and have them bound into the Privacy Graph*. For example a policy that states that data can be accessed by the subject of the record if they have been level-3 authenticated by these trusted services. These are NOT evaluated by privacy pipes but are evaluated and inside Privacy Domains producing the necessary Trust Credentials.

These Trust Criteria may be of any format and schema and may be included by reference or value. To be captured inside a Privacy Graph they need to be in a JSON format or encoded in a format that can be a value in JSON, for example, a string. Once in this format, the data owner can associate these Trust Criteria with the data (1) in the data plane using the privacy_pipe_request see below, and (2) in the management plane using the PN Data Model see below.

```json
// The PN Data Trust Criteria object is used to capture the Trust Criteria
{
  "id": "globally unique id",
  "type": ["pn_data_trust_criteria", "custom_type_info"],
  "scope": "graph",
  "trust_criteria": [
    { "type": "some_custom_type", "prop1": "", "prop2": "", "etc": "etc" },
    { "type": "some_custom_type", "value": "encoded policies" }
  ],
}


//
// The optional Privacy_Pipe_Request has a trust_criteria property that can be set to
// necessary trust criteria
//
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "trust_criteria": "<pn_data_trust_criteria>",
  "more_props_removed_for_clairty": "",
}

// The PN Data model allows a party to define a default so that if none are passed
// to the privacy pipe this value is used
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_trust_criteria": "<pn_data_trust_criteria>",
  "more_props_removed_for_clairty": "",
}

//
// A PN Privacy Graph payload containing the trust_criteria.
// Note the PG header and digital signature are removed for clarity.
//
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "trust_criteria": "<pn_data_trust_criteria>",
    "destination_privacy_domain_trust_criteria": "<pn_trust_criteria_statement>",
    "other_props_removed_for_clairty": "",
  },
  "data": []
}

```

## Associating Trust Credentials with the Data for Provenance
Both PN Trust Credentials and other Trust Credentials can be cryptographically bound to the data inside a Privacy Graph and used as Provenance by other parties. The data owner can associate PN Trust Criteria with data (1) in the data plane using the privacy_pipe_request see below, and (2) in the management plane using the PN Data Model see below.

*The PN automatically associates the data source Privacy Domain's PN Trust Credentials with the data* by cryptographically binding them into the Privacy Graph property 'source_privacy_domain_trust_credentials' as non-compact JWTs as shown below. A destination Privacy Domain can control what data flows into it from another Privacy Domains via a reverse proxy Privacy Pipe by defining a set of quality PN Trust Criteria that the source Privacy Domain must have a superset of.

```json
// example Privacy Graph with the source privacy domain's PN Trust Credentials
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "source_privacy_domain_trust_credentials":[
      { "header":{},
        "payload": {
          "pn_jwtid": "<globally unqiue id>",
          "pn_jwt_type": "pn_trust_credential",
          "provenance_attribute_such_as_secure_source": "1",
        },
        "signature":{},
      }
    ],
    "more_props_removed_for_clairty": "",
  },
  "data": []
}
```

The data owner can optionally *associate any Trust Credentials with the data and have them bound into the Privacy Graph*. These Trust Credentials may be of any format and schema and may be included by reference or value. To be captured inside a Privacy Graph they need to be in a JSON format or encoded in a format that can be a value in JSON, for example, a string. Once in this format, the data owner can associate these Trust Credentials with data (1) in the data plane using the privacy_pipe_request see below, and (2) in the management plane using the PN Data Model see below.


```json
{
  "id": "globally unique id",
  "type": ["pn_data_trust_credentials", "custom_type_info"],
  "trust_credentials": [
    { "type": "some_custom_type", "prop1": "", "prop2": "", "etc": "etc" },
    { "type": "some_custom_type", "value": "encoded tc" }
  ],
}


//
// The optional Privacy_Pipe_Request has a trust_credentials property that can be
// set to the pn_data_trust_credentials object.
//
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "trust_credentials": "<pn_data_trust_credentials>",
  "more_props_removed_for_clairty": "",
}

// The PN Data model allows a party to define a default so that if none are passed
// to the privacy pipe this value is used
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_trust_credentials": "<pn_data_trust_credentials>",
  "more_props_removed_for_clairty": "",
}

//
// A PN Privacy Graph payload containing the trust_credentials.
// Note the PG header and digital signature are removed for clarity.
//
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "trust_credentials": "<pn_data_trust_credentials>",
    "source_privacy_domain_trust_credentials":[],
    "other_props_removed_for_clairty": "",
  },
  "data": []
}


```
