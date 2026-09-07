# Colombian compliance boundary for the initial POS

_Research date: 2026-09-07_

This note defines a product boundary for Velkra v1. It is not legal, tax, or accounting advice. Before deployment, the Store should have a Colombian accountant or tax adviser verify the owner's current RUT, tax responsibilities, and obligation to invoice.

## Decision

Velkra v1 may record operational Sales and produce a clearly identified **internal receipt**, but that receipt must not claim to be a tax invoice or a DIAN-valid document. Velkra must not become the Store's only checkout system until the Store's invoicing status is verified. If the Store is required to invoice, every Sale must also be issued through either a compliant electronic invoice flow or, where applicable, a DIAN-compliant electronic equivalent POS document. V1 should integrate with or run alongside an authorized invoicing solution instead of partially imitating one.

This boundary is deliberately stricter than “no electronic invoicing in v1”: the absence of an invoicing feature does not remove the Store's legal invoicing obligations.

## Why the deployment gate is necessary

- DIAN Resolution 000165 of 2023, articles 5 and 7, requires obligated subjects—including merchants except for expressly excluded subjects—to issue an invoice or equivalent document for each sale when it occurs. Article 8 lists the exceptions; an income threshold by itself is not a general product rule that Velkra can safely apply without the owner's actual tax status. DIAN's consolidated invoicing doctrine likewise says the obligation generally applies to sales and services and that exceptions are the legally enumerated ones. [DIAN Resolution 000165 of 2023](https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf), arts. 5, 7–9; [DIAN Unified Invoicing Concept 0106 of 2022, updated compilation](https://normograma.dian.gov.co/dian/compilacion/docs/concepto_tributario_dian_0000106_2022.htm), section 1.2.2.
- An obligated merchant is generally an electronic invoicer under article 9. An obligated subject may use an electronic equivalent document only for an operation covered by an authorized equivalent-document category; otherwise it must issue an electronic sales invoice (articles 14–16). The electronic POS ticket is one such regulated document, not a synonym for any receipt printed by point-of-sale software. [DIAN Resolution 000165 of 2023](https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf), arts. 9, 14–20.
- The implementation dates for electronic equivalent documents have passed. DIAN now states that obligated invoicers who choose an equivalent document covered by article 15 must replace it with the corresponding electronic equivalent or an electronic invoice. [DIAN electronic equivalent document guidance](https://micrositios.dian.gov.co/sistema-de-facturacion-electronica/documento-equivalente-electronico/); [DIAN current regulatory index](https://micrositios.dian.gov.co/sistema-de-facturacion-electronica/marco-normativo-y-documentacion-tecnica-del-doc-equivalente-electronico/).

Therefore the v1 onboarding checklist must block “Velkra-only checkout” until a qualified adviser records one of these outcomes:

1. the Store is not obligated to invoice and the legal basis has been documented; or
2. the Store is obligated, and Velkra has a defined companion/integration workflow with a compliant invoicing provider or DIAN solution.

The product must not attempt to infer this status from revenue, business size, or a few onboarding answers.

## What the internal receipt may and may not do

The internal receipt may help the Customer and Store understand the operation. It may show Store identification, an internal Sale number, date and time, Sale Lines, quantities, agreed prices, total, payment method, outstanding balance, delivery status, and who recorded the operation.

It should be headed **“Comprobante interno — no es factura de venta ni documento equivalente DIAN”** until it is backed by a compliant fiscal document. The UI and export must avoid:

- calling the receipt “factura”, “factura electrónica”, “documento equivalente electrónico” or “tiquete POS DIAN”;
- presenting it as support for taxes, costs, deductions, or deductible VAT;
- showing invented DIAN authorization ranges, CUFE/CUDE, QR validation, digital signature, or “validado por la DIAN” status;
- implying that sharing a PDF or printing a thermal receipt satisfies invoicing duties; or
- silently replacing the receipt after synchronization in a way that loses the original operational record.

Those terms and artifacts belong to regulated documents with prescribed generation, numbering, tax, signature, validation, delivery, and technical requirements. For example, articles 19 and 20 require a DIAN-authorized range for an electronic POS ticket, CUDE, QR/DIAN lookup, digital signature, tax fields, product identification, and DIAN validation/delivery. [DIAN Resolution 000165 of 2023](https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf), arts. 18–20. DIAN also confirms that an electronic POS equivalent can be issued regardless of transaction value, so Velkra must not revive the former five-UVT product assumption. [DIAN Concept 001563 of 2024](https://www.dian.gov.co/Contribuyentes-Plus/Documents/06-CONCEPTO-001563-int-146-04032024.pdf).

## Records v1 should preserve

Even though v1 is not the fiscal system of record, its immutable operational history should preserve enough linkage to reconcile both systems:

- internal Sale identifier, creation/confirmation timestamps, user and originating device;
- every Sale Line's product, quantity, Reference Price, agreed unit price, taxes only if sourced from the fiscal provider, and total;
- cash/credit form, payment method, installments, delivery and cancellation/correction history;
- Customer identity captured for Credit Sales or Layaways, subject to the privacy rules below;
- the companion fiscal-document type, provider, number, issue timestamp, DIAN validation status, CUFE/CUDE or provider identifier, and original payload/representation pointer when one exists;
- explicit states such as `not_required`, `pending_issue`, `issued`, `rejected`, `contingency`, and `needs_review`, without treating local sync completion as DIAN validation; and
- append-only links between corrections and the original Sale. DIAN requires notes to correct or cancel electronic equivalent documents and prohibits reusing the cancelled number. [DIAN Resolution 000165 of 2023](https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf), art. 22.

The compliant invoicing solution remains responsible for the legally required document, retention, transmission, validation, and delivery. Velkra's copy is a reconciliation aid, not a substitute.

## Offline operation is not automatic fiscal contingency

Velkra can accept operational Sales while its own cloud connection is unavailable, but it must not label ordinary local-first operation as an authorized fiscal contingency. Resolution 000165 distinguishes failures attributable to DIAN from failures of the invoicer:

- during a DIAN validation outage, the electronic document may be issued without prior validation and must later be transmitted under the prescribed process; and
- during the invoicer's own technology problem, prescribed physical contingency documents and later electronic generation/transmission apply. For an electronic equivalent document, article 37 requires transmission within 48 hours after service restoration and specifies different procedures depending on whose technology failed.

[DIAN Resolution 000165 of 2023](https://www.dian.gov.co/normatividad/Normatividad/Resoluci%C3%B3n%20000165%20de%2001-11-2023.pdf), arts. 33 and 37.

Consequently, a future fiscal integration must expose the provider's real contingency status and deadlines. V1 may queue an operational Sale but must display its fiscal issuance as pending and alert the Administrator; it cannot promise that background synchronization alone makes a late document compliant.

## Suppliers and Purchases

Velkra may record Purchases and Payables operationally, but it must retain any invoice or equivalent-document reference received from the Supplier. If the Store buys from a subject not obligated to invoice and needs the acquisition to support tax costs, deductions, or deductible VAT, DIAN requires a regulated electronic support document; a Velkra Purchase record is not that document. This capability is outside v1 and should be handled by the Store's accounting/fiscal solution. [DIAN Unified Invoicing Concept 0106 of 2022, updated compilation](https://normograma.dian.gov.co/dian/compilacion/docs/concepto_tributario_dian_0000106_2022.htm), section 4.1.1.

## Customer data boundary

Names, phone numbers, addresses, and identity documents are personal data. The Store, as the party deciding why and how those data are used, must provide the purpose, obtain and preserve prior informed authorization when no legal exception applies, allow access/correction/deletion where applicable, and protect the data against loss and unauthorized access. Velkra should therefore support a concise privacy notice/authorization record, purpose-limited collection, role-based access, correction requests, export, retention rules, encryption and audit history. It should not collect a Customer for anonymous paid Sales merely because the software can.

[Colombian Law 1581 of 2012](https://www.suin-juriscol.gov.co/viewDocument.asp?id=1684507), arts. 4, 8–12 and 17; [SIC guidance on prior, express and informed authorization](https://sedeelectronica.sic.gov.co/publicaciones/boletin-juridico/boletin/no-es-una-buena-practica-utilizar-mecanismos-para-que-tenga-el-mismo-efecto-que-la-persona-acepte-o-no-acepte).

## Specification consequences

The Velkra v1 specification should include these acceptance boundaries:

1. Internal receipts always carry the non-fiscal disclaimer unless returned by a compliant fiscal integration.
2. Deployment onboarding records the externally verified invoicing status and blocks unsupported Velkra-only checkout.
3. Operational and fiscal status are separate; offline/local sync never implies DIAN issuance or validation.
4. Sales preserve append-only corrections and optional fiscal-document linkage.
5. Purchases preserve Supplier document references but do not claim to create DIAN support documents.
6. Customer identity is optional for ordinary paid Sales and protected by an explicit data-treatment workflow when collected.
7. Electronic invoicing, electronic equivalent-document generation, tax determination/filing, payroll, and formal accounting remain out of scope. If the business needs any of them to trade compliantly, Velkra must integrate with a compliant service or remain a companion operational ledger.

## Revalidation trigger

Recheck this boundary immediately before production rollout and whenever the Store's RUT/responsibilities, legal form, tax regime, invoicing provider, or DIAN rules change. DIAN's regulatory index already records amendments to Resolution 000165, including Resolutions 000008 and 000119 of 2024 and 000202 of 2025, so implementation must use then-current requirements rather than freezing this research note into code. [DIAN invoicing regulatory index](https://micrositios.dian.gov.co/sistema-de-facturacion-electronica/normatividad/).
