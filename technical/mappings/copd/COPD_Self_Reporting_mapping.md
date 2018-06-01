## COPD Self Reporting to openEHR FLAT JSON PROMS Mapping guidance

1st June-2018
V1.1.0

### Introduction

This document describes the mappings between in input COPD Self Reporting XML document and an output openEHR Marand FLAT JSON document.

When describing the JSON target we have used 'handlebar' {{}} type notation to show how the source datapoint is mapped/substituted into the JSON document.

e.g.

```json
		"ctx/health_care_facility|id": "{{SendingFacility}}",
```

where `SendingFacility` is the name of the source XML tag.

In a number of cases the incoming value e.g a score or level must be mapped to an internal openEHR code, when the mapping tables will be shown.


#### XML Source: `patient`

```XML
<patient ident='99928'>
```

`patient` is not mapped as this demographics information is not held in the openEHR CDR.
`Patient/ident` should be used to identify the patient and then look up their openEHR `ehrId` via the CDR API.

#### XML source: `protocol/testid`
```xml
     <protocol name='NHS Health Check' testid='106'>
```

This is not mapped but can be done easily so if required

JSON Header...

##### Example mapping:
```json
"ctx/language": "en",
"ctx/territory": "GB",
"ctx/composer_name": "COPD Self Reporting",
"ctx/time": "{{Now()}}",
"ctx/id_namespace": "HOSPITAL-NS",
"ctx/id_scheme": "HOSPITAL-NS",
"ctx/health_care_facility|name": "COPD Self Reporting",
"ctx/health_care_facility|id": "9091",  
```

** Notes:**

All of the values on the header should be hardwired, other than `ctx/time`, which should simply record the current dateTime i.e the time that this document was authored in [ISO8061](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a003169814.htm) format e.g.

2015-07-1509:03:36Z
####  XML source: `line`
```xml
<line question='Systolic blood pressure' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='mmHg' value='149' valuetype='Numeric' />
```

General approach



  #### Systolic and Diastolic Blood pressure - `readcode` = `'2469'` or `'246A'`:

	```xml
	<line question='Systolic blood pressure' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='mmHg' value='149' valuetype='Numeric' />
	<line question='Diastolic blood pressure' alerted='Normal' codevalue='' term='O/E - Diastolic BP reading' readcode='246A' snomed='254076013' resdate='2015-07-15 09:03:36Z' itemindex='0' units='mmHg' value='86' valuetype='Numeric' />
```
	##### JSON target:  

	```json
	"copd_self_reported_observations/blood_pressure/history_origin" :"{{resdate to ISO format}}"
	"copd_self_reported_observations/blood_pressure/any_event:0/systolic|magnitude": "{{value where readcode= `2469`}}",
	"copd_self_reported_observations/blood_pressure/any_event:0/systolic|unit": "mm[Hg]",
	"copd_self_reported_observations/blood_pressure/any_event:0/diastolic|magnitude":  "{{value where readcode= `2469`}}",
	"copd_self_reported_observations/blood_pressure/any_event:0/diastolic|unit": "mm[Hg]",
	```

	** Notes: **

  Where units=mmHg these should be mapped to "mm[Hg]" which is official UCUM unit format.

	`resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
	e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

	#### Body temperature - `readcode` =`'246A'`:

	```xml
	<line question='Temperature' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
	```
	##### JSON target:  

	```json
	"copd_self_reported_observations/body_temperature/history_origin" : "{{resdate to ISO format}}",
	"copd_self_reported_observations/body_temperature/any_event:0/temperature|magnitude": {{units}},
	"copd_self_reported_observations/body_temperature/any_event:0/temperature|unit": "Cel"
	```

	** Notes: **

  `resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

	Where units=mmHg these should be mapped to "Cel" which is official UCUM unit format.

	#### Heart rate - `readcode` =`'246A'`:

	```xml
	<line question='Heart rate' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
	```
	##### JSON target:  

	```json
	"copd_self_reported_observations/heart_rate/history_origin" : "{{resdate to ISO format}}"
	"copd_self_reported_observations/heart_rate/any_event:0/heart_rate|magnitude":  {{value}},
	"copd_self_reported_observations/heart_rate/any_event:0/heart_rate|unit": "/min",
	```

	** Notes: **

	`resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

  Unit is hardwired to '/min'.

	#### O2 sats - `readcode` =`'246A'`:

	```xml
	<line question='O2 sats' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
	```
	##### JSON target:  

	```json
	"copd_self_reported_observations/o2sat/history_origin" : "{{resdate}}",
	"copd_self_reported_observations/o2sat/any_event:0/o2sat|numerator": {{value}},
	"copd_self_reported_observations/o2sat/any_event:0/o2sat|denominator": 100,
	```

	** Notes: **

	`resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
	e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

	Denominator is hardwired to 100.
