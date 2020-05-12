# ICAREdata Submission Procedures

version 0.1

## Introduction

### Purpose

This document outlines the submission procedures and data expectations for the ICAREdata™ Study.  The interface leverages [FHIR Messaging](http://www.hl7.org/implement/standards/fhir/messaging.html)

### Target Audience and Expectations

The primary audience for this document are those that are directly involved in the ICAREdata Study data submission process, though the content is relevant for anyone involved in the study itself.

This document relies heavily on the reader having a basic understanding of the HL7 FHIR specification. The document does not provide an overview of the FHIR specification and will only explain the use of FHIR within the realm of the ICAREdata Study.

### Scope

This document outlines the supported data formats for the system, data content expectations, submission protocols and authorization mechanisms.

This document does not describe the procedures or mechanisms for integrating with an ICAREdata Study site's electronic health record (EHR) or other systems.  The document also does not describe any processing required to transform the data into a format that is compatible with the data submission server. These tasks are site-specific in nature and as such are outside of the scope of this document.

### Assumptions

1. This document assumes FHIR Specification [R4](http://hl7.org/fhir/timelines.html). It is envisioned that other FHIR versions may be supported at some time in the future, when that support is added this documentation will be updated to reflect that fact.

## Technical Approach

The ICAREdata Study submission infrastructure will use a FHIR messaging based approach.

### Message Definition

The icare-data-study message is a FHIR Message sent by a participating ICAREdata Study site to report data gathered about a clinical trial subject to the ICAREdata submission infrastructure.

#### Message Structure

The general structure of the icare-data-study message is presented below. Please note that the constraints for mCODE and ICAREdata Study Implementation Guides (IG) entries are not included below as they are described in detail within each of the aforementioned IGs.

- Bundle
  - id -- _identifier of the bundle_
  - type "message"
  - entry
    - fullUrl -- *sender-generated uuid*
    - resource
      - MessageHeader
        - id -- _identifier of the message_
        - timestamp -- _time the message was sent_
        - event
          - system "urn::ICAREdataStudy" -- _namespace for icare-data-submission message event code_
          - code  "icare-data-submission" -- _code value to denote a icare-data-submission message_
        - source
          - endpoint -- _the address to which responses to this message should be sent_
        - focus
          - reference  -- _reference to a Bundle containing the information for the clinical trial and subject data for this message_
  - entry
    - fullUrl  -- *sender-generated uuid that identifies this entry*
    - resource -- _FHIR Bundle that contains the data about the clinical trial and subject_
      - Bundle
        - id -- *sender-generated uuid that identifies this entry*
        - type -- _the type of Bundle "collection"_
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _mCODE CancerDiseaseStatus entry for primary disease activity_
            - Observation
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _mCODE CancerDiseaseStatus entry for metastatic disease activity_
            - Observation
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _ICAREdata Study TreatmentPlanChange entry_
            - CarePlan
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _mCODE PrimaryCancerCondition entry_
            - Condition
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _mCODE SecondaryCancerCondition entry_
            - Condition
        - entry
          - fullUrl -- *sender-generated uuid that identifies this entry*
          - resource -- _mCODE Patient entry_
            - Patient

See the FHIR Specification for the data type definitions and optional elements for the resources used in the icare-data-submission message.

- [Bundle](http://www.hl7.org/implement/standards/fhir/bundle.html)
- [MessageHeader](http://www.hl7.org/implement/standards/fhir/messageheader.html)

See the Implementation Guides for mCODE and ICAREdata Study for constraints of the individual data entries.

- [mCODE](http://build.fhir.org/ig/HL7/fhir-mCODE-ig/)
- [ICAREdata Study](http://build.fhir.org/ig/standardhealth/fsh-icare/)

The message header will contain a single data element that reference a Parameters resource that appears in an entry element in the message. The Parameters resource provides information that about the clinical trial and the clinical trial subject that the message pertains too.

#### Example Message

- Example ICAREdata Submission message: [JSON](icare-data-submission-request-json-example.md)

### icare-data-study response

When the ICAREdata Study submission infrastructure accepts and processes a data submission request, the response message will contain a Message Header with response code value, "ok".

When the ICAREdata Study submission infrastructure rejects a data submission request, the MessageHeader will have the response code value, "fatal-error" and reference an [OperationOutcome](http://hl7.org/fhir/DSTU2/operationoutcome.html) resource contained in another entry in the bundle.

The OperationOutcome will have issue severity value "error" or "fatal" and an issue code value from the value set, [issue-type](http://www.hl7.org/implement/standards/fhir/valueset-issue-type.html). The issue code provides ICAREdata Study clients with information about why the submission infrastructure did not process the request. Human-friendly text that complements the issue code may also be provided in an issue details element.

#### Message Structure

- Bundle
  - id -- _identifier of the bundle_
  - type "message"
  - entry
    - fullUrl -- *sender-generated uuid that identifies this entry*
    - resource
      - MessageHeader
        - id -- _identifier of the message_
        - timestamp -- _time the message was sent_
        - event
          - system "urn::ICAREdataStudy" -- _namespace for ICAREdata message event code_
          - code  "icare-data-study" -- _code value to denote a submission request message_
        - response
          - identifier -- _identifier of the request message for which this is a response; specifically the ID request message's MessageHeader.id_
          - code "ok | fatal-error" -- _See [Response Type](http://www.hl7.org/implement/standards/fhir/valueset-response-code.html) in the FHIR Specification_
          - details -- _included only if the response code is fatal-error_
            - reference -- _reference to an OperationOutcome resource that contains information about the error_
  - entry -- _Included only if the ICAREdata Study submission system rejected the submission request_
    - fullUrl -- *sender-generated uuid that identifies this entry*
    - resource
      - OperationOutcome
        - id -- _identifier of this OperationOutcome resource; referenced above in MessageHeader_
        - issue
          - severity -- "fatal | error" -- _See [Operation Outcome](https://www.hl7.org/fhir/operationoutcome-examples.html) in the FHIR Specification_
          - code -- _value from the value set,  [issue-type](http://www.hl7.org/implement/standards/fhir/valueset-issue-type.html)_
          - details -- _Optional_
            - text -- _Human-friendly description of the error or reason the ICAREdata system is rejecting the request_

See the FHIR Specification for the data type definitions and optional elements for the  resources used in this message.

- [Bundle](http://www.hl7.org/implement/standards/fhir/bundle.html)
- [MessageHeader](http://www.hl7.org/implement/standards/fhir/messageheader.html)
- [OperationOutcome](http://www.hl7.org/implement/standards/fhir/operationoutcome.html)

#### Example Messages

- Successful Request: [JSON](icare-data-submission-response-json-example-01.md)
- Rejected Request 1: [JSON](icare-data-submission-response-json-example-02.md)

## Message Delivery Mechanism

Section [3.4.3](https://www.hl7.org/fhir/messaging.html#process) of the FHIR Specification defines a process for handling FHIR Messages via the $process-message operation. The specification outlines both synchronous and asynchronous means of submitting the messages to a FHIR server for processing. For the purposes of data submission in the ICAREdata Study infrastructure, only synchronous messaging will be supported and as such the submission server will expect messages to be submitted following the this guidance in the FHIR specification.

Future versions of the submission infrastructure may support asynchronous messaging and at such time this documentation will be updated to reflect those procedures.

### Content Type

While the FHIR specification supports both JSON and XML as primary data protocols the ICAREdata Study submission infrastructure will support only JSON based interactions.

### Security Implementation Guidance

For the purposes of authorizing site clients to the ICAREdata infrastructure the submission services will support the protocols outlined in the SMART Backend Services: Authorization Guide. The submission infrastructure will be acting as a FHIR server for this purpose and will implement the server side of the protocol.  Site clients will be required to implement the client side of the protocol.

### Client registration

Clients for participating sites will be manually registered in the ICAREdata infrastructures OAuth2 server. Sites participating in the ICAREdata Study will be provided with the client identifiers and authentication/authorization credentials.

- [SMART Backend Services: Authorization Guide](https://build.fhir.org/ig/HL7/bulk-data-export/authorization/index.html)

## Resources

- [FHIR Specification](http://hl7.org/fhir/index.html)
- [FHIR Messaging](http://www.hl7.org/implement/standards/fhir/messaging.html)
