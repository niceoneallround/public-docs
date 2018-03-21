# Trust Criteria and Trust Credentials

Trust Criteria is a Trust Model term for any business, compliance, or privacy policy that controls access a resource they are associated with.

Trust Credential is a Trust Model term for any signed document making a claim about a subject.

Trust Criteria and Trust Credentials can be defined at the Privacy Domain level (PN Trust Criteria and PN Trust Credentials) and at the Data Level (Data Trust Criteria and Data Trust Credentials). At the Privacy Domain level they are used to control information flow between Privacy Domains, and at the data level they are used to control access to the data.

## Privacy Domain Level

PN Trust Criteria are used at the Privacy Domain level. They have a known format and schema so that they can be enforced by the Privacy Agent as part of a Privacy Pipe. PN Trust Criteria describe a set of required PN Trust Credentials in terms of attribute and issuers and evaluates to true for a Privacy Domain if that Privacy Domain has a super-set of the required PN Trust Credentials.

PN Trust Criteria can be associated with data and are cryptographically bound into the Privacy Graph that envelops the data owners data by the Privacy Agent. Today the PN Trust Criteria are scoped at the data graph level, in the future this will be type and property.

Privacy Pipes enforce the data owners trusted destination Privacy Domain PN Trust Criteria and data will not be forwarded to any Privacy Domain that does not have a superset of the required PN Trust Credentials.

PN Trust Credentials are used at the Privacy Domain level and can be issued to Privacy Domains by other Privacy Domains. They have a known format and schema so that they can be processed by the Privacy Agent but they can contain any user defined attribute (usually a globally unique URL) that represents the credential and are part of a Trust Model.

A PN Trust Credential is a JWT that that contains the user defined attribute, the subject Privacy Domain, and the issuer Privacy Domain, in the future other issuers will be supported. WebShield provides a command line tool to issue PN Trust Credentials.

```json
//
// PN Trust Criteria
//
{
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
// A PN Privacy Graph containing the PN Trust Criteria destination_privacy_domain_trust_criteria
//
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "destination_privacy_domain_trust_criteria": "<pn_trust_criteria_statement>",
    "more_props_removed_for_clairty": "",
  },
  "data": []
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

The data owner can associate PN Trust Criteria with data
- In the data plane using the privacy_pipe_request.destination_privacy_domain_trust_criteria property
- In the management plane using the PN Data Model.default_destination_privacy_domain_trust_criteria property

```json

// Pipe request
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria object",
}

// Data model
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria>",
}
```

## Data Level

Data Trust Criteria capture the business, compliance and privacy policy semantics that a data owners needs to associate with the data, for example a policy that states that medical record that can be accessed by subject of the record if they have been level-3 authenticated by these trusted services. These are NOT used to determine if data can be forwarded to a Privacy Domain.

 also be defined at the data level and cryptographically bound inside the Privacy Graph. These are NOT used to determine if data can be forwarded to a Privacy Domain but capture the business, compliance and privacy policy semantics that a data owners needs to associate with the data. For example medical record that can be accessed by subject of the record if they have been level-3 authenticated by these trusted services. These may be included by reference or by value.

Data Trust Criteria may be of any format and schema, and may be included by reference or value. To be captured inside a Privacy Graph they need in a JSON format or encoded in a format that can be a value in JSON for example a string. Once in this format they can be captured by a PN Data Trust Criteria Object that has the following format, note there is no scope property as it is assumed this is all managed inside the custom trust criteria in the future if needed this can be added so parties can use PN pattern to associate Data Trust Criteria with types and properties.

```json
// example PN data trust criteria object capturing Data Trust Criteria
{
  "id": "globally unique id",
  "type": ["pn_data_trust_criteria", "custom_type_info"],
  "trust_criteria": [
    { "type": "some_custom_type", "prop1": "", "prop2": "", "etc": "etc" },
    { "type": "some_custom_type", "value": "encoded policies" }
  ],
}
```

The data Trust Criteria are cryptographically bound into a Privacy Graph in the 'data_trust_criteria' property either by value or by reference (id)

```json
// example Privacy Graph
{
  "@context": "<pn jsonld context>",
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "data_trust_criteria": {},
    "destination_privacy_domain_trust_criteria": "<pn_data_trust_criteria>",
    "more_props_removed_for_clairty": "",
  },
  "data": []
}
```
The data owner can associate data Trust Criteria with data at
- In the data plane using the privacy_pipe_request.data_trust_criteria property
- In the management plane using the PN Data Model.default_data_trust_criteria property

```json

// Pipe request
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "data_trust_criteria": "<pn_data_trust_criteria>",
  "more_props_removed_for_clairty": "",
}

// Data model
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_data_trust_criteria": "<pn_data_trust_criteria>",
  "more_props_removed_for_clairty": "",
}
```

# Provenance

In this document Data Provenance is general term that covers the data's Provenance, Lineage and Descriptive information.

Provenance can be defined at the Privacy Domain level (PN Trust Credentials) and at the Data Level (Data Provenance) and are both cryptographically bound into a Privacy Graph.

## Privacy Domain Level

PN Trust Credentials can be used to make provenance claims about Privacy Domains and PN Data Models. A Privacy Agent always cryptographically binds the data source Privacy Domain PN Trust Credentials into the Privacy Graph in the 'source_privacy_domain_trust_credentials' property as non compact JWTs as shown below.

A destination Privacy Domain can control what data flows into it from another Privacy Domains via a reverse proxy Privacy Pipe by defining a set of quality PN Trust Criteria that the source Privacy Domain must have a superset of.

```json
// example Privacy Graph with the source privacy domain PN Trust Credentials
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
    "data_provenance": "<pn_data_provenance object>",
    "more_props_removed_for_clairty": "",
  },
  "data": []
}
```

## Data Level

The Data level Provenance may be of any format and schema but to be captured inside a Privacy Graph it needs to be either in JSON or encoded in a format that can be a value in JSON for example a string or other. Once in this format it can be captured by a PN Data Provenance Object that has the following format, note there is no scope property as it is assumed this is all managed inside the Data Trust Criteria in the future if needed this can be added so parties can use PN pattern to provenance with types and properties.

```json
{
  "id": "globally unique id",
  "type": ["pn_data_provenance", "custom_type_info"],
  "provenance": [
    { "type": "some_custom_type", "prop1": "", "prop2": "", "etc": "etc" },
    { "type": "some_custom_type", "value": "encoded provenance" }
  ],
}
```

The Data Provenance are cryptographically bound into a Privacy Graph in the 'data_provenance' property either by value or by reference.

```json
{
  "id": "<pg id>",
  "type": "privacy_graph",
  "metadata": {
    "type": "privacy_graph_metadata",
    "data_provenance": "<pn_data_provenance>",
    "more_props_removed_for_clairty": "",
  },
  "data": []
}
```

The data owner can associate Data Provenance with data
- In the data plane using the privacy_pipe_request.data_provenance property
- In the management plane using the PN Data Model.default_data_provenance property

```json

// Pipe request
{
  "id": "<request_id>",
  "type": "privacy_pipe_request",
  "data": [],
  "data_provenance": "<pn_data_provenance>",
  "more_props_removed_for_clairty": "",
}

// Data model
{
  "id": "<id>",
  "type": "pn_datamodel",
  "default_data_provenance": "<pn_data_provenance>",
  "more_props_removed_for_clairty": "",
}
```
