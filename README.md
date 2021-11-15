<a href="https://yeovilhospital.co.uk/">
	<img alttext="Yeovil District Hospital Logo" src="https://github.com/Fdawgs/ydh-logos/raw/HEAD/images/ydh-full-logo-transparent-background.svg" width="480" />
</a>

# Yeovil District Hospital NHS Foundation Trust - Migrating to ITK3 Transfer of Care FHIR messages

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat)](https://github.com/prettier/prettier)

> Yeovil District Hospital NHSFT's ITK3 Transfer Of Care Migration Work Logs

## Intro

### Purpose

This repo outlines the steps that have been taken to migrate from Kettering XML to NHS Digital's [Transfer of Care](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/transfer-of-care-specification-versions) specification and [ITK3](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/itk3-test-harness/itk3-messaging-distribution-specification-versions) FHIR resource components for sending clinical documents via [MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

### Background

Yeovil District Hospital NHS Foundation Trust manages [Yeovil District Hospital](https://yeovilhospital.co.uk/) (YDH) in Yeovil, Somerset.

Yeovil is situated next to the Dorset county border and the hospital sees a sizeable amount of patients from Dorset every year (14.92% of A&E attendances and 15% of inpatients in 2020 were from Dorset).

Every encounter with a patient generates a document, usually an inpatient discharge summary, emergency discharge summary, or outpatient clinic letter.
These documents are generated as PDFs and then converted to HTML (using [Docsmith](https://github.com/Fdawgs/docsmith)) for them to be placed in an XML payload using the Kettering XML format.
The Kettering XML payloads are sent to a patient's primary care provider system electronically via [NHS Digital's MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

Dorset GP Practices use [TPP's SystmOne](https://tpp-uk.com/products/), whilst Somerset GP Practices use [EMIS](https://www.emishealth.com/products/emis-web/) for their primary care system.

EMIS has no issue receiving these documents as HTML and presenting them in their system.

SystmOne however, strips a large amount of the HTML elements out (including all `<img>` elements) and then saves the documents as `.tif` files, which have led to several issues, including:

-   Paragraphs overlapping each other, leading to unreadable documents
-   All images removed from documents (cannot send scan results)
-   The bottom third of each page is missing, and all the content with it

This all impacts patient care, with the issue of missing content being the most severe.

Multiple Dorset-based GP practices have reported one or more of these issues since 2018-10-09.
Attempts to rectify this on YDH's end proved unsuccessful and so this was raised with Phil Mabey, Steve Howes, and Andy Hadley at [Dorset CCG](https://www.dorsetccg.nhs.uk/) on 2020-06-19 for them to raise with TPP.
At the time of writing (2021-10-25), this has still not been resolved.

It was [noticed in an article in Digital Health](https://www.digitalhealth.net/2020/08/clinical-patient-discharge-summaries-soon-to-be-sent-electronically-to-gps/) that TPP was trialling the support of FHIR resources in Dorset. We are hoping that migrating to NHS Digital's Transfer of Care spec and ITK3 FHIR resource profiles will resolve the aforementioned issues and, if not, at least future proof us for the eventual full migration to the HL7 FHIR standards.

#### Overlapping Paragraph Examples

<img alttext="A document printed from TPP's SystmOne, with overlapping text" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_overlapping_1.png" width="480">

<img alttext="Another document printed from TPP's SystmOne, with overlapping text" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_overlapping_2.png" width="480">

#### Removed Images Examples

What was sent to a Dorset-based GP practice:

<img alttext="A document generated at YDH, with the hospital logo present" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_missing_image_1_1.png" width="480">

How the document was saved and presented as by TPP's SystmOne, note the missing hospital logo:

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

The [Transfer of Care specification](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/transfer-of-care-specification-versions) defines FHIR resources that can be sent inside ITK3 structured bundles, such as eDocuments.

YDH will be looking at utilising the following Transfer of Care FHIR resources:

-   Acute Inpatient Discharge
-   Emergency Care Discharge
-   Outpatient Clinic Letter

### MESH Integration

The ITK3 Transfer of Care FHIR resource bundles will need to be sent to GP surgeries in Somerset and Dorset, as well as other NHS trusts, via [MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

YDH currently uses the [Java-based MESH client](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh/compare-mesh-services), which utilises `.dat` and `.ctl` Kettering XML files, however NHS Digital also provides a [web-based RESTful API](https://digital.nhs.uk/developer/api-catalogue/message-exchange-for-social-care-and-health-api).

[Documentation on how to send ITK3 FHIR resource bundles via MESH](https://developer.nhs.uk/apis/itk3messagedistribution-2-9-0/mesh.html) is non-existent, and NHS Digital has been contacted (on 2021-11-15) to clarify on the following queries:

-   Whether the web-based RESTful API is required for sending FHIR payloads, or if the Java-based client can be used
-   What content-types the API POST endpoints can accept, as ideally this would be sent as `application/json`

### Supplier Support

#### EMIS

#### TPP's SystmOne

## License

`ydh-toc-itk3` is licensed under the [MIT](./LICENSE) license.
