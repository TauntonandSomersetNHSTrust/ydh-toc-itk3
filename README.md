<a href="https://yeovilhospital.co.uk/">
	<img alttext="Yeovil District Hospital Logo" src="https://github.com/Fdawgs/ydh-logos/raw/HEAD/images/ydh-full-logo-transparent-background.svg" width="480" />
</a>

# Yeovil District Hospital NHS Foundation Trust - Migrating to Transfer of Care and ITK3 FHIR messages

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat)](https://github.com/prettier/prettier)

> Yeovil District Hospital NHSFT's Transfer Of Care and ITK3 Migration Work Logs

## Intro

### Purpose

This repo outlines the steps that have been taken to migrate from Kettering XML to NHS Digital's [Transfer of Care](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/transfer-of-care-specification-versions) specification and [ITK3](https://digital.nhs.uk/services/interoperability-toolkit/developer-resources/itk3-test-harness/itk3-messaging-distribution-specification-versions) FHIR resource profiles for sending clinical documents via [MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

### Background

Yeovil District Hospital NHS Foundation Trust manages [Yeovil District Hospital](https://yeovilhospital.co.uk/) in Yeovil, Somerset.

Yeovil is situated next to the Dorset county border and the hospital sees a sizeable amount of patients from Dorset every year (14.92% of all A&E attendances and 15% of all inpatients in 2020 were from Dorset).

Every encounter with a patient generates a document, usually an inpatient discharge summary, emergency discharge summary, or outpatient clinic letter.
These documents are generated as PDFs and then converted to HTML (using [Docsmith](https://github.com/Fdawgs/docsmith)) for them to be placed in an XML payload using the Kettering XML format.
The Kettering XML payloads are sent to a patient's primary care provider system electronically via [NHS Digital's MESH](https://digital.nhs.uk/services/message-exchange-for-social-care-and-health-mesh).

Dorset GP Practices use [TPP's SystmOne](https://tpp-uk.com/products/), whilst Somerset GP Practices use [EMIS](https://www.emishealth.com/products/emis-web/) for their primary care system.

EMIS has no issue receiving these documents as HTML and presenting them in their system.

SystmOne however, strips a large amount of the HTML tags out (including all the image tags) and then saves the documents as `.tiff` files which have led to a number of issues, including:

-   Paragraphs overlapping each other, leading to unreadable documents
-   All images removed from documents (cannot send scan results)
-   The bottom third of each page is missing, and all the content with it

This all impacts patient care, with the issue of missing content being the most severe.

<img alttext="A document printed from TPP's SystmOne, with overlapping text" src="https://raw.githubusercontent.com/Fdawgs/ydh-toc-itk3/master/docs/images/example_overlapping.png" width="480">

Multiple Dorset-based GP practices have reported back one or more of these issues since 2018-10-09.
Attempts to rectify this on YDH's end proved unsuccessful and so this was raised with [Dorset CCG](https://www.dorsetccg.nhs.uk/) on 2020-06-19 for them to raise with TPP.
At the time of writing (2021-09-27), this has still not been resolved.

It was [noticed in an article in Digital Health](https://www.digitalhealth.net/2020/08/clinical-patient-discharge-summaries-soon-to-be-sent-electronically-to-gps/) that TPP was trialling the support of FHIR resources in Dorset, and so we are hoping that migrating to NHS Digital's Transfer of Care spec and ITK3 FHIR resource profiles will resolve the aforementioned issues and, if not, at least future proof us for the eventual full migration to the HL7 FHIR standards.

## License

`ydh-toc-itk3` is licensed under the [MIT](./LICENSE) license.
