# Evaluating the Appropriateness of the OMOP Common Data Model for Radiology Research: A Comprehensive Architectural and Implementation Analysis

## 1. The Philosophical and Architectural Paradigm: Research vs. Operational Data Management

To evaluate the appropriateness of the Observational Medical Outcomes Partnership (OMOP) Common Data Model (CDM) as a data repository for radiology research, one must first deconstruct the fundamental dichotomy between operational healthcare data management and the requirements of observational research. The user’s query, which juxtaposes "research" purposes against "hospital data management," touches upon the core existential mandate of the OHDSI (Observational Health Data Sciences and Informatics) community. The OMOP CDM was not designed to facilitate the transactional operations of a hospital—such as scheduling, billing, or immediate clinical decision support—but rather to standardize the structure and content of observational data to enable efficient analyses that produce reliable evidence.1

### 1.1 The Divergence of Purpose: Suitability for Analysis

Operational systems, including Electronic Health Records (EHR) and Picture Archiving and Communication Systems (PACS), are architected for data retrieval specificity and transactional integrity. A radiology order in a hospital system is a transient operational state, linked to a billing code (CPT) and a DICOM accession number, primarily serving the workflow of image acquisition and interpretation. In contrast, the OMOP CDM is governed by the principle of "suitability for purpose," where that purpose is the generation of evidence through statistical characterization, population-level effect estimation, and patient-level prediction.1

This philosophical divergence has profound implications for data modeling. Operational data is often siloed, proprietary, and functionally heterogeneous, varying significantly from one organization to the next.1 Data collected for provider reimbursement or direct patient care often lacks the semantic consistency required for large-scale analytics. For instance, the concept of "blood glucose" or a "CT Chest" might be represented in dozens of different proprietary formats across a single health network.1 The OMOP CDM resolves this by transforming these disparate formats into a common physical schema and, critically, mapping local vernaculars to a Standardized Vocabulary.2

For radiology research, this transformation is pivotal. Research in this domain is increasingly moving beyond simple utilization metrics (e.g., "how many CTs were performed?") to complex phenotypic questions (e.g., "what is the correlation between specific radiomic features of lung nodules and five-year survival rates?"). The operational data model of a PACS, which isolates imaging metadata in DICOM headers, is ill-suited for this type of longitudinal outcomes research. The OMOP CDM, particularly with its extensions, provides the relational fabric to weave imaging data into the broader clinical tapestry of diagnoses, drug exposures, and laboratory measurements.3

### 1.2 The Evolution of the Model: From Administrative Claims to Clinical Granularity

Historically, the OMOP CDM was optimized for administrative claims data, focusing on billing codes and enrollment periods. However, the trajectory of the model has bent sharply toward clinical granularity to support EHR and registry data.5 This evolution is evident in the transition from CDM v4 to v5 and v6, which introduced tables like MEASUREMENT and NOTE\_NLP to capture structured and unstructured clinical findings.2

The necessity for this shift is underscored by the limitations of claims data for clinical inquiry. Administrative codes often lack the precision to distinguish between the *ordering* of a test and the *result* of a test. In radiology, this distinction is critical. A claim for a "Mammogram" tells a researcher that the procedure occurred; it does not reveal the BI-RADS score, the breast density, or the presence of microcalcifications. The OMOP CDM’s "suitability for purpose" in radiology research is therefore contingent upon its ability to capture these deeper phenotypes, a capability that has been significantly bolstered by the recent development of the Medical Imaging Extension.7

### 1.3 Data Protection and Governance in the Research Context

A primary advantage of the OMOP CDM for research, as opposed to raw hospital data dumps, is its embedded approach to data protection and governance. The model is designed to facilitate federated research networks where patient-level data remains behind the local firewall, and only aggregate statistics or answers to queries are shared.9 This "privacy by design" is essential for multi-institutional radiology studies, which often involve massive datasets of sensitive imaging information.

The CDM dictates that all data jeopardizing patient privacy—such as names and precise birthdates—be removed or generalized during the ETL (Extract, Transform, Load) process.2 For example, the PERSON table stores year\_of\_birth, month\_of\_birth, and day\_of\_birth as separate integer fields, allowing for the easy truncation of precise dates if required by local governance protocols.10 This structural consideration makes the OMOP CDM inherently more appropriate for collaborative research than a raw replica of a hospital database, which would require extensive ad-hoc de-identification before any cross-site analysis could occur.9

## 2. The Structural Foundation: Core Domains and Identity Management

To assess the appropriateness of the schema, one must examine the core table structures that define the OMOP CDM. These tables form the skeleton upon which radiology data must be draped. The rigidity of these structures ensures interoperability but imposes specific constraints that the user has identified as potential hurdles: specifically, the management of patient identity and the definition of clinical visits.

### 2.1 The Person-Centric Architecture

The OMOP CDM is strictly person-centric. The PERSON table is the root of the hierarchy, serving as the central identity management system for the entire database.10

* **Single Record Constraint:** The schema mandates that each individual human being has exactly one record in the PERSON table. This record contains time-invariant demographics such as year of birth, gender (gender\_concept\_id), and race (race\_concept\_id).10
* **Foreign Key Integrity:** Almost every other clinical event table—VISIT\_OCCURRENCE, CONDITION\_OCCURRENCE, MEASUREMENT, PROCEDURE\_OCCURRENCE—contains a person\_id foreign key that must reference a valid entry in the PERSON table.12

This architecture is highly advantageous for research because it enforces a longitudinal view of the patient. In a hospital system, a patient might have different "Medical Record Numbers" (MRN) for their inpatient stay versus their outpatient radiology visit. The OMOP ETL process forces the reconciliation of these disparate identifiers into a single person\_id, creating a unified longitudinal record.14 However, this requirement becomes a significant engineering challenge when merging disparate datasets that lack a common Master Patient Index (MPI), a specific issue raised by the user and detailed in Section 4.

### 2.2 The Vocabulary and Concept System

The "Standardized Vocabularies" are the semantic engine of the OMOP CDM. Unlike a hospital database that might store the text string "CT Chest w/o Contrast" or the local code "RAD101", the OMOP CDM requires these source values to be mapped to a concept\_id found in the Standardized Vocabularies (e.g., SNOMED, LOINC, RxNorm).1

* **Source vs. Standard:** The schema preserves the original data fidelity through \_source\_value columns (e.g., procedure\_source\_value) while simultaneously enabling standardized analysis through \_concept\_id columns (e.g., procedure\_concept\_id).13
* **Implication for Radiology:** This allows researchers to query for "Computed Tomography of Chest" using a single standard SNOMED concept, automatically retrieving records that were originally coded in CPT-4, HCPCS, or local vernaculars across different hospitals. This semantic harmonization is a prerequisite for large-scale "research" and is a key differentiator from operational data models.3

### 2.3 The Domain-Based Organization

The CDM organizes data into domains based on the *nature* of the clinical event, not the source of the data.

* **Conditions:** Diagnoses and symptoms go to CONDITION\_OCCURRENCE.
* **Procedures:** Interventions (surgeries, imaging) go to PROCEDURE\_OCCURRENCE.
* **Measurements:** Lab tests and quantitative findings go to MEASUREMENT.17

This domain-driven approach ensures that a diagnosis of "Pneumonia" is always found in the CONDITION\_OCCURRENCE table, regardless of whether it came from an admitting diagnosis code, a discharge summary, or a radiology report finding. This predictable structure simplifies the development of standardized analytical tools.19 However, for radiology, this scattering of data—where the *scan* is a Procedure but the *finding* is a Condition or Measurement—can be operationally complex to assemble during ETL, necessitating the specialized extensions discussed in Section 3.

## 3. The Medical Imaging Extension (MI-CDM): Bridging the Gap

The standard OMOP CDM tables (PROCEDURE\_OCCURRENCE, MEASUREMENT) were historically insufficient for deep radiology research. They could capture the fact that a CT scan occurred and perhaps a few derived measurements, but they failed to capture the rich acquisition metadata (e.g., slice thickness, kVp, magnet strength) and the hierarchical structure of DICOM data (Study -> Series -> Instance).7 To address this, the OHDSI Medical Imaging Working Group developed the Medical Imaging Extension (MI-CDM), which renders the schema significantly more "appropriate" for radiology research than the base model alone.

### 3.1 Evolution: From R-CDM to MI-CDM

The initial effort, known as the Radiology-CDM (R-CDM), focused primarily on radiological imaging. This has since evolved into the Medical Imaging Extension (MI-CDM) to encompass all DICOM-based specialties, including ophthalmology and pathology.7 This evolution reflects the growing need to link algorithmically generated measurements (radiomics, AI features) into the OMOP data model to harness deeper phenotypes.8

### 3.2 The Image\_occurrence Table

The Image\_occurrence table is the structural cornerstone of the extension. It creates a standardized repository for imaging events, mirroring the "Series" level of the DICOM hierarchy. By introducing this table, the CDM can now store technical metadata that is irrelevant to general clinical care but vital for imaging research.20

Table Structure and Research Utility:

The Image\_occurrence table is designed to link the clinical world (Person, Visit) with the imaging world (DICOM files).

| **Field Name** | **Function & Research Utility** |
| --- | --- |
| **image\_occurrence\_id** | **Primary Key.** Unique identifier for the imaging event series. Essential for linking features back to specific scans.8 |
| **person\_id** | **Foreign Key.** Links the image to the patient's longitudinal record, enabling correlation with drugs and outcomes.20 |
| **visit\_occurrence\_id** | **Foreign Key.** Links the image to the healthcare encounter (Inpatient/Outpatient), providing clinical context.21 |
| **image\_occurrence\_date** | Captures the precise date of acquisition. Critical for establishing temporal causality in longitudinal studies.8 |
| **wadors\_uri / local\_path** | Stores the pointer to the actual pixel data in the PACS/VNA. This turns the CDM into a catalog for retrieving images for AI training.8 |
| **modality\_concept\_id** | Stores the standardized concept for the modality (e.g., CT, MRI, US), enabling easy cohort stratification.22 |
| **anatomical\_site\_concept\_id** | Stores the SNOMED concept for the body part examined (e.g., "Lung", "Brain"), standardizing diverse DICOM body part tags.20 |

This table allows a researcher to define a cohort not just by "Patients who had a CT," but "Patients who had a Chest CT with 1mm slice thickness".8 This capability is a quantum leap over standard hospital data querying capabilities.

### 3.3 The Image\_feature Table

While Image\_occurrence tracks the event, the Image\_feature table tracks the *content* derived from that event. This is the primary repository for "computational phenotypes" and radiomic features.8

Provenance and Semantic Linkage:

The Image\_feature table is designed with a One-to-Many relationship to Image\_occurrence, reflecting the fact that a single image series can yield thousands of features (e.g., texture analysis, geometric measurements).23

* **Concept Mapping:** Features are mapped to extended vocabularies. RadLex is used for radiological findings (e.g., "ground glass opacity"), while LOINC is used for quantitative measurements.20
* **Algorithm Provenance:** The table supports type\_concept\_id fields to distinguish the origin of the feature. A "tumor size" measured by a radiologist (manual) is semantically distinct from a "tumor size" calculated by a deep learning algorithm (automated). The MI-CDM supports this distinction, which is critical for validating AI models.24
* **Linkage to Clinical Domains:** The table includes foreign keys (image\_feature\_event\_id, image\_feature\_event\_field\_concept\_id) that allow it to link to MEASUREMENT or OBSERVATION tables. This allows features to be stored as standard clinical measurements while retaining their specific imaging provenance.20

### 3.4 Case Study Validation: The ProCAncer-I Project

The utility of these tables has been validated in large-scale projects such as the ProCAncer-I project, which collected data from 9,822 patients across multiple sites. In this study, clinical data was converted to the OMOP CDM oncology extension, while DICOM metadata was converted to the MI-CDM extension. The project successfully instantiated the Image\_occurrence and Image\_feature tables to support the analysis of prostate cancer progression, proving that the schema is robust enough for complex, multi-modal oncology research.26

## 4. The Identity Crisis: Managing Person\_ID Overlap in Disparate Datasets

The user explicitly raises the issue of person\_id overlap across disparate radiology datasets. This is a pervasive challenge in "federated" or "network" research where data from Site A and Site B must be combined into a single OMOP instance for analysis. In a hospital operational environment, a Master Patient Index (MPI) server would resolve these identities using probabilistic matching of names and social security numbers. In the de-identified, disparate world of OMOP research, such identifiable data is often unavailable or legally restricted, leading to unique architectural challenges.

### 4.1 The Collision Problem and the Person\_ID Constraint

In the OMOP CDM, person\_id is a unique integer serving as the primary key of the PERSON table. It is the spine of the entire model.10

* **The Scenario:** Radiology Dataset A (e.g., an MRI center) has a patient with internal ID 101. Radiology Dataset B (e.g., a separate hospital) also has a patient with internal ID 101.
* **The Conflict:** If these datasets are naively merged, the person\_id uniqueness constraint will be violated. Even if the constraint is technically managed, the system will conflate two different individuals into a single record, corrupting the clinical history.27

The OHDSI community consensus is that resolving patient overlap (identifying that Patient X at Site A is the same as Patient Y at Site B) requires linking data through identifiable information stored *outside* the OMOP CDM. If this external linkage is impossible (due to privacy or lack of common identifiers), the CDM cannot magically resolve the overlap.28

### 4.2 Implementation Strategy: ID Namespacing (Offsetting)

To utilize the OMOP CDM as a central repository for these disparate datasets without creating collisions, the standard engineering practice is **ID Offsetting** (often called Namespacing or Partitioning) during the ETL process.27

**The Algorithm:**

1. **Assign Site Offsets:** The ETL architect assigns a distinct, non-overlapping integer range to each data source.
   * *Site A Range:* 1 to 999,999,999
   * *Site B Range:* 1,000,000,000 to 1,999,999,999
2. **Transform IDs:** During the ETL, the local identifier is transformed.
   * Site A Patient 101 $\rightarrow$ person\_id 101
   * Site B Patient 101 $\rightarrow$ person\_id 1,000,000,101
3. **Unified Schema:** Both records can now coexist in the PERSON table. All downstream tables (MEASUREMENT, IMAGE\_OCCURRENCE) use these new offset IDs as foreign keys.

Research Implications:

This strategy renders the schema "appropriate" for storage and basic analysis because it prevents technical failure. However, it introduces a critical analytical limitation: Fragmentation of the Longitudinal Record. The CDM will treat Site A's "101" and Site B's "101" as two completely different people.

* *Impact on Prevalence:* Prevalence counts will be accurate (assuming the two 101s are different people).
* *Impact on Incidence:* If the *same* person visits both sites, their history is split. A cancer diagnosis at Site B might look like a "new" case even if they were treated at Site A.
* *Mitigation:* Research cohorts must be defined with the understanding that "Person" refers to a "Person-Site" entity unless external tokenization (privacy-preserving record linkage) is used.28

### 4.3 Traceability: Person\_Source\_Value and Metadata

To maintain the integrity of the research repository, it is vital to preserve the link to the original disparate source.

* **person\_source\_value:** This column in the PERSON table should store the original, unaltered identifier from the source system (e.g., "SiteB\_101"). This allows researchers to trace data anomalies back to the source file.10
* **care\_site\_id:** The CARE\_SITE table should be populated to reflect the originating radiology center. By linking every VISIT\_OCCURRENCE to a care\_site\_id, researchers can filter or stratify analysis by data source, essentially creating "virtual" sub-databases within the single CDM repository.29

### 4.4 Managing De-identification in Merged Datasets

When merging disparate datasets, privacy risks increase. The OHDSI community utilizes de-identification methods such as the Hripcsak method or simple time-shifting to protect patient privacy while preserving temporal relationships.32

* **Time-Shifting:** All dates for a specific person\_id are shifted by a random number of days (e.g., -14 to +14).
* **Consistency:** In a merged repository, it is crucial that if a patient *is* linked across sites (via an external MPI), the same time-shift offset is applied to both sources. If they are treated as separate person\_ids (via namespacing), independent shifts are acceptable.32

## 5. The Contextual Void: Populating Visit\_Occurrence Without Explicit Encounters

The second major implementation hurdle identified by the user is the difficulty of populating visit\_occurrence\_id foreign keys when explicit visit contexts are missing. This is typical in radiology-only datasets (e.g., a dump from a PACS or a RIS) which record the *procedure* but often lack the admission/discharge data found in an ADT (Admission-Discharge-Transfer) system.

### 5.1 The Role of the Visit in OMOP Analysis

In the OMOP CDM, the VISIT\_OCCURRENCE table represents the span of time a person engages with the healthcare system (e.g., Inpatient Visit, Outpatient Visit, ER Visit).6

* **The "Hub" Function:** The visit\_occurrence\_id acts as a central hub. It is a foreign key in DRUG\_EXPOSURE, CONDITION\_OCCURRENCE, PROCEDURE\_OCCURRENCE, and MEASUREMENT.12
* **Analytical Dependency:** Many standardized analytical tools (such as the OHDSI ATLAS cohort builder) rely on the Visit concept to define time-at-risk windows or to stratify by care setting (e.g., "Community acquired pneumonia" implies pneumonia diagnosed in an *outpatient* or *ER* visit, not during an *inpatient* stay).

### 5.2 The Challenge of "Orphaned" Events

Technically, the OMOP schema defines visit\_occurrence\_id as a nullable field in most clinical tables. The DDL (Data Definition Language) for PostgreSQL, for example, does not impose a NOT NULL constraint on this column in the MEASUREMENT table.12

* **The Risk:** While technically allowed, creating "orphaned" events (measurements without a visit) degrades the utility of the research repository. It renders the data invisible to analyses that require visit context (e.g., utilization studies, cost analyses, or visit-anchored timelines).33

### 5.3 The Solution: The "Constructed Visit" ETL Strategy

To make radiology data "appropriate" for research, the standard best practice is to **construct** or **infer** visits from the available procedure dates. This approach is widely used in the OHDSI community for converting claims and registry data where visit groupings are not explicit.34

The "One Day" Logic:

The most common heuristic for "departmental" data is the "One Day" roll-up logic.

| **Step** | **Description** | **Logic** |
| --- | --- | --- |
| **1. Sort** | Organize all timestamped events for a patient chronologically. | ORDER BY person\_id, event\_date |
| **2. Cluster** | Group events that occur close together (typically on the same calendar day) into a single cluster. | IF (Current\_Date - Previous\_Date) <= 1 Day THEN Same\_Visit 36 |
| **3. Generate** | Create a new VISIT\_OCCURRENCE record for each cluster. | visit\_start\_date = Min(event\_dates); visit\_end\_date = Max(event\_dates) |
| **4. Assign Concept** | Assign a generic Visit Concept ID if the specific type is unknown. | Use Concept 9202 (Outpatient Visit) or 581477 (Radiology Visit) as a default.12 |
| **5. Link** | Update the clinical tables with the newly generated ID. | UPDATE Image\_Occurrence SET visit\_occurrence\_id = New\_Visit\_ID.35 |

Synthetic ID Generation:

Since these visits do not exist in the source, the ETL must generate a synthetic primary key for the VISIT\_OCCURRENCE table. A common approach is to use a hash of the person\_id and the visit\_start\_date, or a simple auto-incrementing integer sequence that is unique across the dataset.30

Research Integrity Note:

When using this method, it is crucial to populate the visit\_type\_concept\_id with a concept that explicitly states "Visit derived from EHR record" or "Visit inferred from procedure" (e.g., Concept ID 32024 or similar). This preserves the provenance, informing future researchers that this "Visit" is an analytical construct, not a confirmed hospital admission.6

### 5.4 Visit\_Detail vs. Visit\_Occurrence

The user also queries the difficulty of populating visit\_detail\_id. In OMOP CDM v5.3+, VISIT\_DETAIL represents micro-encounters (e.g., a specific stay in the ICU or Radiology department) within a macro VISIT\_OCCURRENCE (the entire hospital stay).37

* **Complexity:** Populating VISIT\_DETAIL requires understanding the hierarchy of the encounter.
* **Recommendation:** For radiology-only datasets where the broader hospital context is missing, populating VISIT\_DETAIL is often unnecessary and adds complexity without value. The VISIT\_OCCURRENCE is the mandatory table for standard analysis. VISIT\_DETAIL should be reserved for datasets where the distinct movement of the patient between departments is known and analytically relevant.12

## 6. Semantic Interoperability: The Vocabulary Layer

The structural appropriateness of the schema is useless without semantic consistency. For radiology research, this involves mapping the "Tower of Babel" of local procedure codes and text descriptions into standard OHDSI vocabularies.

### 6.1 Integrating RadLex and DICOM

The OHDSI Medical Imaging Working Group has prioritized the integration of **RadLex** (Radiology Lexicon) and **DICOM** terminology into the OMOP Vocabulary.7

* **RadLex:** This ontology is essential for describing *findings* (e.g., "Pulmonary Nodule," "Ground Glass Opacity"). While SNOMED covers general clinical terms, RadLex provides the specific granularity needed for imaging phenotypes.20
* **DICOM Attributes:** Standard attributes (e.g., Modality, Anatomic Region) are mapped to OMOP Concepts. For example, the DICOM tag for "Body Part Examined" (0018,0015) is mapped to SNOMED anatomical concepts in the Image\_occurrence table.20

### 6.2 The Mapping Burden

A significant challenge in converting radiology data is the mapping of local procedure codes (e.g., "CT CHEST W CONTRAST - PROTOCOL A") to standard Concepts (e.g., CPT or SNOMED).

* **Usagi:** OHDSI provides a tool called **Usagi** to assist in this process. It uses fuzzy string matching to suggest mappings from source descriptions to standard concepts.39
* **Post-Coordination:** Radiology findings are often complex (e.g., "Large mass in the left upper lobe"). OMOP standardly uses pre-coordinated concepts, but the MEASUREMENT and Image\_feature tables allow for some post-coordination (linking a "Mass" concept to a "Left Upper Lobe" concept via value\_as\_concept\_id or modifier\_concept\_id).40

## 7. The ETL Ecosystem: From Design to Validation

The "appropriateness" of the OMOP CDM is also a function of the mature ecosystem of tools available to implement it. Unlike a bespoke hospital SQL database, the OMOP CDM comes with a suite of open-source software designed to standardize the ETL process.

### 7.1 The White Rabbit and Rabbit-in-a-Hat

Before a single line of code is written, the OHDSI tool **White Rabbit** scans the source data (e.g., the CSV dumps from the radiology system) to generate a report on field values and distributions. This is critical for understanding the "dirty" reality of the source data.41

* **Rabbit-in-a-Hat:** This tool takes the scan report and allows the researcher to graphically map source tables to CDM tables. It generates a specification document that serves as the blueprint for the ETL.43

### 7.2 Data Quality Dashboard (DQD)

Once the data is loaded, the **Data Quality Dashboard (DQD)** runs over 3,500 automated checks against the database. It validates foreign key integrity (e.g., "Does every visit\_occurrence\_id in IMAGE\_OCCURRENCE exist in VISIT\_OCCURRENCE?"), plausibility (e.g., "Is the scan date after the patient's death?"), and conformance.12 This automated validation is a massive differentiator for "research" repositories, providing a level of trust in the data quality that ad-hoc hospital databases rarely possess.

## 8. Comparative Analysis: OMOP vs. FHIR for Research

To fully contextualize the appropriateness of OMOP, it is helpful to briefly contrast it with HL7 FHIR (Fast Healthcare Interoperability Resources), which is often mentioned in hospital data management contexts.

* **FHIR:** Designed for *data exchange* (moving a record from Point A to Point B). It is highly nested (JSON), operational, and document-centric.46
* **OMOP:** Designed for *data analysis* (asking questions of the whole population). It is relational, flattened, and optimized for SQL querying.
* **Research Verdict:** While FHIR is excellent for getting data *out* of the hospital system, OMOP is the superior schema for the *repository* itself when the goal is aggregate analysis or cohort study. The "flattening" of complex nested structures into tables like MEASUREMENT and IMAGE\_OCCURRENCE makes analytical queries orders of magnitude faster and simpler to write.9

## 9. Conclusion

The OMOP Common Data Model is unequivocally appropriate for radiology research purposes, arguably more so than raw hospital data management schemas, provided that specific architectural strategies are employed. It transforms the chaotic, siloed, and heterogeneous landscape of operational imaging data into a standardized, patient-centric evidence engine.

The specific challenges raised by the user—person\_id overlap and missing visit contexts—are well-understood "implementation patterns" rather than fatal flaws.

1. **Identity:** By employing **ID Offsetting** and leveraging person\_source\_value, researchers can merge disparate datasets into a unified repository while avoiding collisions, accepting the trade-off of longitudinal fragmentation in the absence of a Master Patient Index.
2. **Context:** By employing **Constructed Visit Logic** in the ETL layer, isolated radiology events can be upgraded into structured VISIT\_OCCURRENCE records, integrating them fully into the OHDSI analytical network.
3. **Granularity:** By adopting the **Medical Imaging Extension (MI-CDM)**, the schema expands to accommodate the high-fidelity metadata of DICOM, bridging the gap between pixel data and clinical outcomes.

Ultimately, adopting OMOP CDM moves the radiology research enterprise from a model of "data scavenging" (parsing diverse, messy headers for every new study) to a model of "evidence generation" (querying a standardized, quality-assured repository). This shift, enabled by the schema's rigid structure and semantic discipline, is the defining characteristic of high-quality observational research.

#### Works cited

1. Standardized Data: The OMOP Common Data Model - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/data-standardization/>
2. documentation:cdm:single-page [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:single-page>
3. Feasibility and utility of applications of the common data model to multiple, disparate observational health databases - PMC - NIH, accessed November 21, 2025, <https://pmc.ncbi.nlm.nih.gov/articles/PMC4457111/>
4. Exploring the potential of OMOP common data model for process mining in healthcare, accessed November 21, 2025, <https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0279641>
5. Transforming and evaluating electronic health record disease phenotyping algorithms using the OMOP common data model: a case study in heart failure | JAMIA Open | Oxford Academic, accessed November 21, 2025, <https://academic.oup.com/jamiaopen/article/4/3/ooab001/6127557>
6. documentation:cdm:visit\_occurrence [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:visit_occurrence>
7. OHDSI Medical Image WG - GitHub Pages, accessed November 21, 2025, <https://ohdsi.github.io/ImageWG/>
8. Development of Medical Imaging Data Standardization for Imaging Based Observational Research: OMOP Common Data Model Extension - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2023/10/6-Park-BriefReport.pdf>
9. Seamless EMR data access: Integrated governance, digital health and the OMOP-CDM, accessed November 21, 2025, <https://informatics.bmj.com/content/31/1/e100953>
10. documentation:cdm:person [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:person>
11. OMOP CDM v5.3 - GitHub Pages, accessed November 21, 2025, <https://ohdsi.github.io/CommonDataModel/cdm53.html>
12. OMOP CDM v5.4 - GitHub Pages, accessed November 21, 2025, <https://ohdsi.github.io/CommonDataModel/cdm54.html>
13. documentation:cdm:data\_model\_conventions [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:data_model_conventions>
14. Combining two or more data sources in a single OMOP database: pros and cons, accessed November 21, 2025, <https://forums.ohdsi.org/t/combining-two-or-more-data-sources-in-a-single-omop-database-pros-and-cons/19201>
15. OMOP Common Data Model - GitHub Pages, accessed November 21, 2025, <https://ohdsi.github.io/CommonDataModel/>
16. Data Model Conventions, accessed November 21, 2025, <https://ohdsi.github.io/CommonDataModel/dataModelConventions.html>
17. documentation:cdm:procedure\_occurrence [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:procedure_occurrence>
18. documentation:cdm:measurement [Observational Health Data Sciences and Informatics] - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:cdm:measurement>
19. Chapter 4 The Common Data Model | The Book of OHDSI, accessed November 21, 2025, <https://ohdsi.github.io/TheBookOfOhdsi/CommonDataModel.html>
20. Development of Medical Imaging Data Standardization for Imaging-Based Observational Research: OMOP Common Data Model Extension - NIH, accessed November 21, 2025, <https://pmc.ncbi.nlm.nih.gov/articles/PMC11031512/>
21. Development of Medical Imaging Data Standardization for Imaging Based Observational Research : OMOP CDM Extension - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2023/08/APAC-10-Kyulee-Jeon_MI-CDM.pdf>
22. Development of Medical Imaging Data Standardization for Imaging-based Observational Research: OMOP CDM Extension - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2024/04/OHDSI_Paper_Presentation_WYPark.pdf>
23. Development of Medical Imaging Data Standardization for Imaging-Based Observational Research: OMOP Common Data Model Extension | springermedizin.de, accessed November 21, 2025, <https://www.springermedizin.de/development-of-medical-imaging-data-standardization-for-imaging-/26701740>
24. OHDSI Medical Imaging Working Group, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2023/02/OHDSI_Imaging_2023_02.pdf>
25. Development of the Medical Imaging Extension for OMOP-CDM - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2022/10/26-SengChan-You_Development-of-the-Medical-Imaging-Extension-for-OMOP-CDM_2022symposium-Seng-Chan-You.pdf>
26. MI-Common Data Model: Extending Observational Medical Outcomes Partnership-Common Data Model (OMOP-CDM) for Registering Medical Imaging Metadata and Subsequent Curation Processes | JCO Clinical Cancer Informatics - ASCO Publications, accessed November 21, 2025, <https://ascopubs.org/doi/10.1200/CCI.23.00101>
27. Best practices for merging two CDM schemas? - Developers - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/best-practices-for-merging-two-cdm-schemas/17678>
28. Overlap/duplications between network sites - Researchers - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/overlap-duplications-between-network-sites/15228>
29. Anyone storing multiple data sources in one CDM? - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/anyone-storing-multiple-data-sources-in-one-cdm/1849>
30. Scalable logic to create custom visit\_ids - Implementers - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/scalable-logic-to-create-custom-visit-ids/14380>
31. Overlap/duplications between network sites - #2 by Christian\_Reich - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/overlap-duplications-between-network-sites/15228/2>
32. De-Identify the CDM Data - Developers - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/de-identify-the-cdm-data/12252>
33. Claims conundrum: all NULLs in VISIT\_OCCURRENCE\_ID in drug exposure table, accessed November 21, 2025, <https://forums.ohdsi.org/t/claims-conundrum-all-nulls-in-visit-occurrence-id-in-drug-exposure-table/2546>
34. Visit Occurrence | Janssen CDM Documentation, accessed November 21, 2025, <https://ohdsi.github.io/ETL-LambdaBuilder/OPTUM_PANTHER/Optum_Panther_Visit_Occurrence.html>
35. How to generate visit\_occurrence\_ids and relate to other clinic data tables - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/how-to-generate-visit-occurrence-ids-and-relate-to-other-clinic-data-tables/15559>
36. Visit\_occurrence - Tutorial-ETL, accessed November 21, 2025, <https://ohdsi.github.io/Tutorial-ETL/cdm_synthea_v1/Visit_occurrence.html>
37. Usage and implementation of visit\_occurrence and visit\_details - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/usage-and-implementation-of-visit-occurrence-and-visit-details/21266>
38. Breaking data silos: incorporating the DICOM imaging standard into the OMOP CDM to enable multimodal research - Oxford Academic, accessed November 21, 2025, <https://academic.oup.com/jamia/article/32/10/1533/8206314>
39. OMOP Conversion Process | OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2024/06/7.-OMOP-Conversion-Process.pdf>
40. Understanding image feature, observation, and measurement tables - OHDSI Forums, accessed November 21, 2025, <https://forums.ohdsi.org/t/understanding-image-feature-observation-and-measurement-tables/24102>
41. ETL Conversion Training - TTRN – CITRIS International Summer Institute, accessed November 21, 2025, <https://summerinstitute.faculty.ucdavis.edu/wp-content/uploads/sites/621/2021/03/Session-4b-OMOP-ETL-Conversions-03262021.pdf>
42. OMOP Common Data Model Extract, Transform & Load Tutorial - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2019/09/OMOP-Common-Data-Model-Extract-Transform-Load.pdf>
43. How to extract transform and load observational data? - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2014/07/Beijing2015.pdf>
44. ETL creation best practices - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/web/wiki/doku.php?id=documentation:etl_best_practices>
45. OHDSI/CommonDataModel: Definition and DDLs for the OMOP Common Data Model (CDM) - GitHub, accessed November 21, 2025, <https://github.com/OHDSI/CommonDataModel>
46. OMOP and FHIR Data Comparison - OHDSI, accessed November 21, 2025, <https://www.ohdsi.org/wp-content/uploads/2022/10/39-Andrey_Soares_OMOPvFHIR_2022Symposium-Lisa-S.pdf>