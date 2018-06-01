## COPD Self Reporting to openEHR FLAT JSON PROMS Mapping guidance

1st June-2018
V1.1.0

### Introduction

This document describes the mappings between in input COPD Self Reporting XML document and an output openEHR Marand FLAT JSON document.

When describing the JSON target we have used 'handlebar' {{}} type notation to show how the source datapoint is mapped/substituted into the JSON document.

e.g.

```json
	"copd_self_reported_observations/heart_rate/any_event:0/heart_rate|magnitude":  {{value}},
```

where `value` is the value of of the source `value` attribute.

#### Outstanding issues

- We do not have an example of an actual COPD xml instance so have been working a little blind in terms of readcode mappings and other tags in the incoming xml but have made a best guess.

- We have pulled in the alerted tag as a simple textual comment. It would be possible to do something smarter if required but the use of this attribute is not clear


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

Target JSON Header

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

**Notes:**

All of the values on the header should be hardwired, other than `ctx/time`, which should simply record the current dateTime i.e the time that this document was authored in [ISO8061](http://support.sas.com/documentation/cdl/en/lrdict/64316/HTML/default/viewer.htm#a003169814.htm) format e.g. `2015-07-15T09:03:36Z`


####  XML source: `patient/protocol/line`

The target archetype / JSION row will depend on the `readcode` attribute in the incoming `line`.

#### Systolic and Diastolic Blood pressure


 ##### `line` attributes
 `readcode`: `2469` or `246A`

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
	"copd_self_reported_observations/blood_pressure/any_event:0/comment": "Alerted: {{Alerted}}",
```
** Notes: **

- `resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

-   Where `units=mmHg` these should be mapped to "mm[Hg]" which is official UCUM unit format.

- The value of `alerted` should be carried in the `comment` element prepended with
	'Alerted' e.g.		`alerted='Normal'` => ``.../comment": "Alerted: Normal",``

#### Body temperature

##### `line` attributes
  `readcode`: `2E3.`

```xml
	<line question='Temperature' alerted='Normal' codevalue='' term='O/E - temperature level' readcode='2E3.' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
```
##### JSON target:  

```json
	"copd_self_reported_observations/body_temperature/history_origin" : "{{resdate to ISO format}}",
	"copd_self_reported_observations/body_temperature/any_event:0/temperature|magnitude": {{units}},
	"copd_self_reported_observations/body_temperature/any_event:0/temperature|unit": "Cel",
	"copd_self_reported_observations/body_temperature/any_event:0/comment": "Alerted: Normal",  
```

 **Notes:**

 - `resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

 - Where units=mmHg these should be mapped to "Cel" which is official UCUM unit format.

 - The value of `alerted` should be carried in the `comment` element prepended with
 'Alerted' e.g. `alerted='Normal'` => `".../comment": "Alerted: Normal"`

#### Heart rate

##### `line` attributes

`readcode`= `24c..`

```xml
	<line question='Heart rate' alerted='Normal' codevalue='' term='Heart rate' readcode='24c..' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
```
##### JSON target:  

```json
	"copd_self_reported_observations/heart_rate/history_origin" : "{{resdate to ISO format}}"
	"copd_self_reported_observations/heart_rate/any_event:0/heart_rate|magnitude":  {{value}},
	"copd_self_reported_observations/heart_rate/any_event:0/heart_rate|unit": "/min",
  "copd_self_reported_observations/heart_rate/any_event:0/comment": "Alerted: {{Alerted}}",  
```

**Notes:**

- `resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

- `unit` is hardwired to '/min'.

- The value of `alerted` should be carried in the `comment` element prepended with
	'Alerted' e.g. `alerted='Normal'` => ``.../comment": "Alerted: Normal",``

#### O2 sats

##### `line` attributes

(no suitable Read code) snomed=`431314004`

```xml
	<line question='O2 sats' alerted='Normal' codevalue='' term='O/E - Systolic BP reading' readcode='2469' snomed='254075012' resdate='2015-07-15 09:03:36Z' itemindex='0' units='cel' value='149' valuetype='Numeric' />
```
##### JSON target:  

```json
	"copd_self_reported_observations/o2sat/history_origin" : "{{resdate}}",
	"copd_self_reported_observations/o2sat/any_event:0/o2sat|numerator": {{value}},
	"copd_self_reported_observations/o2sat/any_event:0/o2sat|denominator": 100,
	"copd_self_reported_observations/o2sat/any_event:0/comment": "Alerted: {{Alerted}}",
````

**Notes:**

-	`resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
	e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

-	`denominator` is hardwired to 100.

- The value of `alerted` should be carried in the `comment` element prepended with
  'Alerted' e.g. `alerted='Normal'` => ``.../comment": "Alerted: Normal",``
