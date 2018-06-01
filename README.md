# Salford Renal PROMS and COPD self monitoring import

## Scope of Work

1. Create openEHR models as a target to receive patient-derived Renal PROMs XML input.
2. Create openEHR models as a target to receive patient-derived COPD management XML input.


### Renal PROMs

1. Analyse the [OPTE_PRO xml input example](technical/mappings/renal/OPTE_PRO_anon_survey.xml) and associated [schema](technical/mappings/renal/Survey.xsd). This is a feed from a system holding Renal medicine-related PROMs information.
2. Create an openEHR template as a target for importing the OPTE_PRO input xml document.
3. Identify existing openEHR archetypes and create new archetypes (reusable where possible) to be used within the openEHR template.
4. Create a mapping guidance document for the input xml to an openEHR FLAT JSON composition.
5. Create associated artefacts e.g. A sample FLAT JSON instance, Marand web template document.

#### Output

##### Reused existing international archetypes
  -  [Report | openEHR-EHR-COMPOSITION.report.v1](models/CKM/remote/org.openehr/archetypes/composition/openEHR-EHR-COMPOSITION.report.v1.adl)
  - [EQ_5D_5L | openEHR-EHR-OBSERVATION.eq_5d_5l.v0](models/local/archetypes/entry/observation/openEHR-EHR-OBSERVATION.eq_5d_5l.v0.adl) (**Not in international CKM for licensing reasons.**)

##### New archetypes
  - [PAM-13 score | openEHR-EHR-OBSERVATION.pam_13.v0](models/local/archetypes/entry/observation/openEHR-EHR-OBSERVATION.pam_13.v0.adl)
  - [POS-S Renal score | openEHR-EHR-OBSERVATION.pos_s_renal.v0](models/local/archetypes/entry/observation/openEHR-EHR-OBSERVATION.pos_s_renal.v0.adl)

It is intended that these will be uploaded/published to the international [openEHR CKM](openehr.org/ckm).

##### New templates
  -  [Salford Renal PROMs Import Report - Salford Renal PROMs Import Report.v0](models/Templates/Salford Renal PROMs Import Report.v0.oet)

##### Mapping guidance / technical artefacts

- [Renal PROMS Mapping Guidance document](technical/mappings/renal/OPTE_PRO_anon_survey_mapping.md)
- [Example target RENAL PROMS Import FLAT JSON **(Sample composition data)** instance](technical/instance/Salford_Renal_PROMS_FLAT_1.json)
- [Salford Renal PROMs Import Report Operational template **(Use for upload to CDR)**](technical/operational/Salford Renal PROMs Import Report.opt)
- [Salford Renal PROMs Import Report - Web template **(Json variant of operational template)**](technical/web_template/Salford_Renal_PROMS__import_template.json)


### COPD Self Reported Observations

1. Analyse the [COPD Self Reported Observations XML](technical/mappings/copd/XML-Example-PMS.TXT) example and associated [schema](technical/mappings/copd/XSD-Example-PMS.txt). This is a feed from a system holding patient device information relating to COPD.
2. Create an openEHR template as a target for importing the COPD input xml document.
3. Identify existing openEHR archetypes and create new archetypes (reusable where possible) to be used within the openEHR template.
4. Create a mapping guidance document for the input xml to an openEHR FLAT JSON composition.
5. Create associated artefacts e.g. A sample FLAT JSON instance, Marand web template document.

#### Output

##### Reused existing international archetypes
  - [Report | openEHR-EHR-COMPOSITION.report.v1](models/CKM/remote/org.openehr/archetypes/composition/openEHR-EHR-COMPOSITION.report.v1.adl)
  - [Pulse oximetry | openEHR-EHR-OBSERVATION.pulse_oximetry.v1](models/CKM/remote/org.openehr/archetypes/entry/observation/openEHR-EHR-OBSERVATION.pulse_oximetry.v1.adl)
  - [Pulse / Heart beat | openEHR-EHR-OBSERVATION.pulse.v1](models/CKM/remote/org.openehr/archetypes/entry/observation/openEHR-EHR-OBSERVATION.pulse.v1.adl)
  - [Blood Pressure | openEHR-EHR-OBSERVATION.blood_pressure.v1](models/CKM/remote/org.openehr/archetypes/entry/observation/openEHR-EHR-OBSERVATION.blood_pressure.v1.adl)
  - [Body temperature| openEHR-EHR-OBSERVATION.body_temperature.v2](models/CKM/remote/org.openehr/archetypes/entry/observation/openEHR-EHR-OBSERVATION.body_temperature.v2.adl)

##### New local archetypes
  - None

##### New templates
  -  [COPD Self Reported Observations - Salford COPD Self Reporting.v0](models/Templates/Salford COPD Self Reporting.v0.oet)

##### Mapping guidance / technical artefacts

- [COPD Self Reported Observations Mapping Guidance document](technical/mappings/copd/COPD_Self_Reporting_mapping.md)
- [Example target COPD Self Reported Observations FLAT JSON instance](technical/instance/Salford_COPD_Self_reporting_FLAT_1.json)
- [COPD Self Reported Observations - Operational template](technical/operational/Salford COPD Self Reporting.opt)
- [COPD Self Reported Observations - Web template](technical/web_template/Salford_COPD_Self_Reporting__import_template.json)
