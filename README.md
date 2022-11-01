<a href="https://yeovilhospital.co.uk/">
	<img alttext="Yeovil District Hospital Logo" src="https://github.com/Fdawgs/ydh-logos/raw/HEAD/images/ydh-full-logo-transparent-background.svg" width="480" />
</a>

# Yeovil District Hospital NHS Foundation Trust - Migrating to ITK3 Transfer of Care FHIR messages

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat)](https://github.com/prettier/prettier)

> Yeovil District Hospital NHSFT's ITK3 Transfer Of Care Migration Work Logs

## Intro

### Purpose

This repo outlines the steps that have been taken to migrate from Kettering XML to NHS Digital's [ITK3](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/itk3-test-harness/itk3-messaging-distribution-specification-versions) [Transfer of Care](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/transfer-of-care-specification-versions) FHIR resource bundles for sending clinical documents via [MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

### Background

Yeovil District Hospital NHS Foundation Trust manages [Yeovil District Hospital](https://yeovilhospital.co.uk/) (YDH) in Yeovil, Somerset.

Yeovil is situated next to the Dorset county border and the hospital sees a sizeable amount of patients from Dorset every year (15.14% of A&E attendances and 13.7% of inpatients in 2021 were from Dorset).

Every encounter with a patient generates a document, usually an inpatient discharge summary, emergency discharge summary, or outpatient clinic letter.
These documents are generated as PDFs and then converted to HTML (using [Docsmith](https://github.com/Fdawgs/docsmith)) for them to be placed in an XML payload using the Kettering XML format.
The Kettering XML payloads are sent to a patient's primary care provider system electronically via [NHS Digital's MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

Dorset GP Practices use [TPP's SystmOne](https://tpp-uk.com/products/), whilst Somerset GP Practices use [EMIS](https://www.emishealth.com/products/emis-web/) for their primary care system.

EMIS has no issue receiving these documents as HTML and presenting them in their system.

SystmOne however, strips a large amount of the HTML elements out (including all `<img>` elements) and then saves the documents as `.tif` files, which has led to several issues, including:

-   [Paragraphs overlapping each other, leading to unreadable documents](#overlapping-paragraph-examples)
-   [All images removed from documents](#removed-images-examples) (cannot send scan results)
-   The bottom third of each page is missing, and [all the content with it](#missing-content-examples)

This all impacts patient care, with the issue of missing content being the most severe.

Multiple Dorset-based GP practices have reported one or more of these issues since 2018-10-09.
Attempts to rectify this on YDH's end proved unsuccessful and so this was raised with Phil Mabey, Steve Howes, and Andy Hadley at [Dorset CCG](https://dorsetccg.nhs.uk/) on 2020-06-19 for them to raise with TPP.
At the time of writing (2022-05-25), this has still not been resolved.

It was [noticed in an article in Digital Health](https://digitalhealth.net/2020/08/clinical-patient-discharge-summaries-soon-to-be-sent-electronically-to-gps/) that TPP was trialling the support of FHIR resources in Dorset. We are hoping that migrating to NHS Digital's ITK3 Transfer of Care FHIR resource bundles will resolve the aforementioned issues and, if not, at least future-proof us for the eventual full migration to the HL7 FHIR standards.

#### Overlapping Paragraph Examples

<img alttext="A document printed from TPP's SystmOne, with overlapping text" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_overlapping_1.png" width="480">

<img alttext="Another document printed from TPP's SystmOne, with overlapping text" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_overlapping_2.png" width="480">

#### Removed Images Examples

What was sent to a Dorset-based GP practice:

<img alttext="A document generated at YDH, with the hospital logo present" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_missing_image_1_1.png" width="480">

How the document was saved and presented by TPP's SystmOne, note the missing hospital logo:

<img alttext="A document printed from TPP's SystmOne, with the hospital logo missing" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_missing_image_1_2.png" width="480">

#### Missing Content Examples

What was sent to a Dorset-based GP practice:

<img alttext="A document printed from TPP's SystmOne, with content present" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_missing_content_1_1.png" width="480">

How the document was saved and presented as by TPP's SystmOne, note the missing text **'clopidogrel 75mg film coated tablets -
Oral'**:

<img alttext="A document printed from TPP's SystmOne, with content missing" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_missing_content_1_2.png" width="480">

Luckily the practice pharmacist noticed it and raised this with us.

## Info and Requirements Gathering

### Specifications

The [ITK3 specification](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/itk3-test-harness/itk3-messaging-distribution-specification-versions) defines the structure and components of a FHIR resource bundle, but not the payload.

The [Transfer of Care (ToC) specification](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/transfer-of-care-specification-versions) defines FHIR resources that can be sent inside ITK3 structured bundles, such as eDocuments.

YDH will be looking at utilising the following Transfer of Care FHIR resources:

-   Acute Inpatient Discharge
-   Emergency Care Discharge
-   Outpatient Clinic Letter

### MESH Integration

The ITK3 Transfer of Care FHIR resource bundles will need to be sent to GP surgeries in Somerset and Dorset, as well as other NHS trusts, via [MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

YDH currently uses the [Java-based MESH client](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/compare-mesh-services), which utilises `.dat` and `.ctl` Kettering XML files, however, NHS Digital also provides a [web-based RESTful API](https://digital.nhs.uk/developer/api-catalogue/message-exchange-for-social-care-and-health-api).

[Documentation on how to send ITK3 FHIR resource bundles via MESH](https://developer.nhs.uk/apis/itk3messagedistribution-2-9-0/mesh.html) is sparse, and NHS Digital was contacted on 2021-11-15 to clarify the following queries:

-   Whether the web-based RESTful API is required for sending FHIR payloads, or if the Java-based client can be used
-   What content-types the API POST endpoints can accept, as ideally this would be sent as `application/json`

The NHS Digital ITOC team (itoc.supportdesk@nhs.net) responded on 2021-11-16 stating that FHIR payloads can be sent by either method and that any type can be POSTed at the API endpoints.
Whilst it would be easy to stay with the Java-based MESH client, for the sake of posterity the web-based RESTful API will be adopted.

### Supplier Support

Dr. Mike Moore, the ToC Project Manager at NHS Digital, was contacted on 2021-11-15 regarding supplier support, and provided the following details:

#### EMIS

EMIS Health needs to deliver Emis Web v9.13.11 to fix a workflow annotation issue before Full Rollout Approval (FRA) can be granted.
The target for FRA or EMIS is 2021-12-22, however, this may be delayed due to holidays.
Post FRA, all of the GP practices using EMIS will need their MESH mailboxes reconfigured to send and receive ToC FHIR messages. This was completed on 2022-01-07.

#### TPP's SystmOne

TPP's SystmOne gained FRA in 2020-08 for inpatient, emergency, and outpatient ToC FHIR resources.
Mike Moore approved FRA for Mental Health Discharge ToC FHIR resources on 2021-11-17.

### Issues With Migration From Current Processes

At present, YDH's documents are generated from supplier systems (Intersystems' TrakCare for inpatient and emergency discharge summaries, BigHand for outpatient letters) as unstructured PDFs.

The hope was that PDFs could be placed into Binary resources inside of the ITK3 structured bundles, however, this is [not allowed](https://developer.nhs.uk/apis/itk3tocedischarge-2-9-0/explore_document_profiles.html) (see Note 1). Instead, they would have to be converted from PDF to HTML, and the resulting markup would need to be parsed to fit into the sections of an [ITK Composition resource](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-ITK-EDIS-Composition-1).

Our neighbouring trust, [Somerset NHS Foundation Trust](https://somersetft.nhs.uk/) (SFT), is also looking to adopt ITK ToC FHIR bundles but will run into a similar issue as their documents are also generated from supplier systems (epro) as unstructured DOCX files.

### Approaching Early Adopters

[Dorset County Hospital NHS Foundation Trust](https://www.dchft.nhs.uk/) (DCH), an early adopter of ITK3 Toc FHIR bundles, was approached on 2021-11-22 to see if they could advise both YDH and SFT on how they achieved adoption. Unfortunately, DCH chose a third-party supplier for their ToC work, who were unwilling to share due to it being achieved with proprietary solutions.

[Devon Partnership NHS Trust](https://dpt.nhs.uk/) (DPT), [Cambridge University Hospitals NHS Foundation Trust](https://cuh.nhs.uk/) (CUH), [Leeds Teaching Hospitals NHS Trust](https://www.leedsth.nhs.uk/) (LTH), and [Oxford University Hospitals NHS Foundation Trust](https://ouh.nhs.uk/) (OUH) are working with suppliers for their ToC implementation.

[Essex Partnership University NHS Foundation Trust](https://eput.nhs.uk/) (EPUT) have built in-house and was happy to discuss their progress. On 2021-12-02 Marc Riding and George Fox from EPUT talked through their development over MS Teams, showing their Mental Health Discharge Summary ToC FHIR Bundle generation process.
Marc shared the code they had developed alongside supporting documentation.

[Northumbria Healthcare NHS Foundation Trust](https://northumbria.nhs.uk/) (NH) has also built in-house and, on 2022-01-18, Richard Leonard and Christopher Rouse talked through their development over MS Teams. NH has used the [Java-based MESH client](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/compare-mesh-services), which utilises `.dat` and `.ctl` Kettering XML files, for sending Transfer of Care documents, and shared their process.

## Accessing the MESH REST API

### Accessing the Development/Test APIs

As noted in the requirements section above, NHS Digital's documentation regarding MESH adoption is confusing.

NHS Digital's ITOC team (itoc.supportdesk@nhs.net) and API Management team (api.management@nhs.net) were contacted on 2022-07-21 in the hope they could clarify what we need to do to access the test MESH APIs.

The API Management team responded on 2022-08-03, pointing us to the [Message Exchange for Social Care and Health (MESH) API](https://digital.nhs.uk/developer/api-catalogue/message-exchange-for-social-care-and-health-api) page that we had already been looking at.

The ITOC team responded on 2022-08-10, directing us to the [Connect to a Path to Live environment](https://digital.nhs.uk/services/path-to-live-environments/connect-to-a-path-to-live-environment) page.

From reviewing both pages, it appears that we need to request a test organisation and test mailbox before gaining access to the test APIs.

The [Connect to a Path to Live environment](https://digital.nhs.uk/services/path-to-live-environments/connect-to-a-path-to-live-environment) page and adjacent [Development Environments](https://digital.nhs.uk/services/path-to-live-environments/development-environment) page state the need for a test organisation, so a test organisation was requested on 2022-08-10. The Test Data team responded on 2022-08-16 with a test organisation (B81008: DR KS TOMMINS' PRACTICE).

A test MESH mailbox was requested via the web form for both RA4 (YDHNSFT) and B81008 (test organisation) on 2022-08-24.

#### Steps

1. Request test organisation to send test messages to, from testdata@nhs.net
1. Fill in Excel spreadsheet sent by testdata@nhs.net and return
1. Await test organisation creation (B81008) by NHS Digital's Test Data team, and the corresponding email
1. [Request test MESH mailbox](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/messaging-exchange-for-social-care-and-health-apply-for-a-mailbox) for live organisation (RA4), with the following Transfer of Care (FHIR) [workflow IDs](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/workflow-groups-and-workflow-ids):
    - TOC_FHIR_EC_DISCH (EC Discharge Summary A and E Report)
    - TOC_FHIR_IP_DISCH (Acute IP and Daycase Discharge)
    - TOC_FHIR_MH_DISCH (MH IP and Daycase Mental Health Inpatient Discharge Summaries)
    - TOC_FHIR_OP_ATTEN (Outpatient Clinic Letter - Outpatient Attendance)
1. [Request test MESH mailbox](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/messaging-exchange-for-social-care-and-health-apply-for-a-mailbox) for the test organisation (B81008), with the following Transfer of Care (FHIR) [workflow IDs](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/workflow-groups-and-workflow-ids):
    - TOC_FHIR_EC_DISCH_ACK
    - TOC_FHIR_IP_DISCH_ACK
    - TOC_FHIR_MH_DISCH_ACK
    - TOC_FHIR_OP_ATTEN_ACK
1. [Register FQDN](https://digital.nhs.uk/services/path-to-live-environments/path-to-live-forms#register-or-modify-your-fqdn-with-the-nhs-dns-service) for test (`local_id`.B81008.api.mesh-client.nhs.uk) and live (`local_id`.RA4.api.mesh-client.nhs.uk) organisation MESH mailboxes

## License

`ydh-toc-itk3` is licensed under the [Open Government Licence v3.0](./LICENSE) license.
