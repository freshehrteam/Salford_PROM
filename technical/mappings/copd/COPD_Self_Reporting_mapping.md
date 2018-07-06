## COPD Self Reporting to openEHR FLAT JSON Mapping guidance

06-July-2018
V2.1.0

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

The target archetype / JSON row will depend on the `question` attribute in the incoming `line`.

 #### Body temperature

##### `line` attributes
  `question`: `TEMPERATURE`

```xml
<line question='TEMPERATURE' alerted='Normal' codevalue='' term='O/E - Core temperature' readcode='2E25' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='�c' value='35.8' valuetype='Numeric' />
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

 - Where units=C these should be mapped to "Cel" which is official UCUM unit format.

 - The value of `alerted` should be carried in the `comment` element prepended with
 'Alerted' e.g. `alerted='Normal'` => `".../comment": "Alerted: Normal"`

#### Heart rate

##### `line` attributes

`question`= `OX_PULSE`

```xml
<line question='OX_PULSE' alerted='Normal' codevalue='' term='' readcode='242' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='Bpm' value='75' valuetype='Numeric' />
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

#### Oxygen Level

##### `line` attributes
`question`: `OXYGEN_LEVEL'
```xml
<line question='OXYGEN_LEVEL' alerted='Normal' codevalue='' term='Blood oxygen saturation' readcode='44YA' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='SpO2' value='93' valuetype='Numeric' />

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


	#### PROM questions

	The other incoming XML `lines` relate to Patient Questions/Answers

	** Notes for ALL Patient questions: **

	- `resDate` must be converted to the ISO8601 format (as used by openEHR and FHIR).
	  e.g. '2015-07-15 09:03:36Z'	-> '2015-07-15T12:09:03:36Z'

	- `resDate` is carried only **once for all of the questions**, it being assumed that all of the questions are answered at the same date and time. You do not actually need to import the date from each `line` but it imay be easier to just overwrite the date already recorded.

	- The value of `alerted` is ignored for question-like responses and is only carried for the numeric device-derived values.

	 ##### `line` attributes
	 `question`: `How are you feeling today or since your last reading?'

	```xml
	<line question='How are you feeling today or since your last reading?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='As well as usual' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/feeling_today_or_since_last_reading": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"

	```



	#### How are you breathing today?

	 ##### `line` attributes
	 `question`: `How are you breathing today or since your last reading?'

	```xml
	<line question='How are you breathing today or since your last reading?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Normal for you' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/breathing_today_or_since_last_reading": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```

	#### Are you using oxygen?

	 ##### `line` attributes
	 `question`: `Are you using oxygen?'

	```xml
	<line question='Are you using oxygen?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='No' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/using_oxygen": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```


	#### Reading taken on oxygen?

	 ##### `line` attributes
	 `question`: `If using oxygen, was your oxygen saturation reading taken on oxygen?'

	```xml
	<line question='If using oxygen, was your oxygen saturation reading taken on oxygen?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='No' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/using_oxygen": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```

	#### Shortness of breath since last reading?

	 ##### `line` attributes
	 `question`: `Since your last reading, have you been experiencing shortness of breath?'

	```xml
	<line question='Since your last reading, have you been experiencing shortness of breath?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Same as last reading' valuetype='Unknown' />

	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/shortness_of_breath_since_last_reading": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```

	#### Are you coughing?

	 ##### `line` attributes
	 `question`: `Are you coughing?`

	```xml
	<line question='Are you coughing?' alerted='Normal' codevalue='' term='' readcode='171' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='No' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/coughing": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```

	#### How is your cough?

	 ##### `line` attributes
	 `question`: `How is your cough?`

	```xml
	<line question='How is your cough?' alerted='Normal' codevalue='' term='' readcode='171' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='No' valuetype='Normal for you' />
	```
	##### JSON target:  

	```json
	  "copd_self_reported_observations/story_history/salford_copd_questions:0/status_of_cough": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```
	#### Are you producing sputum?

	 ##### `line` attributes
	 `question`: `Are you producing sputum?`

	```xml
	<line question='Are you producing sputum?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Normal for you' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
		"copd_self_reported_observations/story_history/salford_copd_questions:0/producing_sputum": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```
	#### Is the colour of your sputum:

	 ##### `line` attributes
	 `question`: `Is the colour of your sputum:`

	```xml
	<line question='Is the colour of your sputum:' alerted='Normal' codevalue='' term='' readcode='1713' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Clear / White' valuetype='Numeric' />
	```
	##### JSON target:  

	```json
		"copd_self_reported_observations/story_history/salford_copd_questions:0/colour_of_sputum": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```
	#### Have you taken your medication including inhalers/nebuliser?

	 ##### `line` attributes
	 `question`: `Have you taken your medication including inhalers/nebuliser?`

	```xml
	<line question='Have you taken your medication including inhalers/nebuliser?' alerted='Normal' codevalue='' term='' readcode='[none]' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Yes' valuetype='Unknown' />
	```
	##### JSON target:  

	```json
		"copd_self_reported_observations/story_history/salford_copd_questions:0/taken_medication": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```
	#### Are your ankles swollen?

	 ##### `line` attributes
	 `question`: `Are your ankles swollen?`

	```xml
	<line question='Are your ankles swollen?' alerted='Normal' codevalue='' term='' readcode='22C2' snomed='' resdate='2017-02-08 07:51:01Z' itemindex='0' units='' value='Moderate but not worse than last reading' valuetype='Numeric' />
	```
	##### JSON target:  

	```json
		"copd_self_reported_observations/story_history/salford_copd_questions:0/swollen_ankle": "{{value}}",
		"copd_self_reported_observations/story_history/salford_copd_questions:0/history_origin" : "{{resdate}}"
	```
