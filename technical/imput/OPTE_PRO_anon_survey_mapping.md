## OPTE_PRO XML to openEHR FLAT JSON PROMS Mapping guidance

30-May-2018
V1.1.0

#### Introduction

This document describes the mappings between in input PPTE-PRO PROMS XML document and an output openEHR Marand FLAT JSON document.

When describing the JSON target we have used 'handlebar' type notation to show how the source datapoint is mapped/substituted into the JSON document.

e.g.

```json
		"ctx/health_care_facility|id": "{{SendingFacility}}",
```

where `SendingFacility` is the name of the source XML tag.

In a number of cases the incoming value e.g a score or level must be mapped to an internal openEHR code, when the mapping tables will be shown.

#### XML Source: `Patient`

`Patient` is not mapped as this demographics information is not held in the openEHR CDR.
`Patient/patientNumber` should be used to identify the patient and then look up their openEHR ehrId via the CDR API.

#### XML source: `SendingExtract`
```xml
	 <SendingExtract>RGU01</SendingExtract>
```

`SendingExtract` is not mapped.

#### XML source: `SendingFacility`
```xml
	 <SendingFacility>RGU01</SendingFacility>
```
##### JSON target:
```json
		"ctx/health_care_facility|id": "{{SendingFacility}}",
```
##### Example mapping:
 ```json
 		"ctx/health_care_facility|id": "RGU01",
		"ctx/health_care_facility|name": "RGU01",
 ```


####  XML source: `ProgramMembership`
```xml
	<ProgramMemberships>
		<ProgramMembership>
			<ProgramName>YourHealth</ProgramName>
			<FromTime>1900-01-01T00:00:00</FromTime>
		</ProgramMembership>
	</ProgramMemberships>
```

##### JSON target:
```json
	"ctx/composer_name": "{{ProgramName}}",
	"ctx/time": "{{FromTime}}",
```

##### Example mapping:
```json
	"ctx/composer_name": "Your Health",
	"ctx/time": "1900-01-01T00:00:00",
```


#### XML source: `Surveys`

```xml
	<Surveys>
		<Survey>
			<SurveyTime>2017-03-06T00:00:00</SurveyTime>
			<SurveyType>
				<Code>PROM</Code>
			</SurveyType>
			<Questions>
       <!-- Snipped - see below for details -->
			</Questions>
			<TypeOfTreatment>Pre-dialysis</TypeOfTreatment>
			<HDLocation></HDLocation>
			<Template>Your Health P1 new 01</Template>
		</Survey>
```

Each survey maps to a different PROM score 'entry' in the openEHR template
`Template` and `HDLocation` are not mapped as we do not have clear examples of content. HDLocation may be 'Haemodialysis location' and could be carried if required.

** `SurveyType/Code = 'PAM'` **
```xml
		<Survey>
			<SurveyTime>2017-03-06T00:00:00</SurveyTime>
			<SurveyType>
				<Code>PAM</Code>
			</SurveyType>
			<Questions>
			 <!-- Snipped for brevity -->
			</Questions>
			<TypeOfTreatment>Pre-dialysis</TypeOfTreatment>
			<HDLocation></HDLocation>
			<Template>Your Health P1 new 01</Template>
			<Scores>
				<Score>
					<ScoreType>
						<Code>PAM_13</Code>
					</ScoreType>
					<Value>32</Value>
					<Level>1</Level>
				</Score>
			</Scores>
		</Survey>
```

##### JSON targets:
  ```json
	"salford_renal_proms_import_report/pam-13:0/history_origin" :"{{SurveyTime}}",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/_name|value" : "{{TypeOfTreatment}}",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/pam_score|magnitude": "{{Score/Value}}",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0//pam_score|unit": "1",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/level|code": "{{Score/Level}}",
	```
Note that  `pam_score|unit` is hardwired to `1` which designates a real number.

PAM-13 `Score/Level`value | Target openEHR `atCode`
----------------|--------------
1               | at0089
2               | at0090
3               | at0091
4               | at0092

##### Example output
  ```json
	"salford_renal_proms_import_report/pam-13:0/history_origin" :"2017-03-06T00:00:00",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/_name|value" : "Pre-dialysis",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/pam_score|magnitude": "32",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/pam_score|unit": "1",
	"salford_renal_proms_import_report/pam-13:0/point_in_time:0/level|code": "at0089",
	```

  #### QuestionType_Code: `'MYHQ1'`:

  ##### JSON target:  
   `"salford_renal_proms_impot_report/pam-13:0/point_in_time:0/a1._responsible_for_taking_care_of_health|code": "{{atCode}}",`


  `'MYHQ1'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0009
  1               | at0005
  2               | at0006
  3               | at0007
  4               | at0008  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ1</Code>
  					</QuestionType>
  					<Response>4</Response>
  				</Question>
  ```

  Maps to

  ```json
  		"salford_renal_proms_import_report/pam-13:0/point_in_time:0/a1._responsible_for_taking_care_of_health|code": "at0008",
  ```

  #### QuestionType_Code: `'MYHQ2'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a2._active_role_is_most_important|code": "{{atCode}}",`


  `'MYHQ2'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0015
  1               | at0016
  2               | at0017
  3               | at0018
  4               | at0019  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ2</Code>
  					</QuestionType>
  					<Response>4</Response>
  				</Question>
  ```

  Maps to

  ```json
   "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a2._active_role_is_most_important|code": "at0019",
  ```

  #### QuestionType_Code: `'MYHQ3'`:

  ##### JSON target:  
   `"salford_renal_proms_import_report/pam-13:0/point_in_time:0/a3._confident_about_preventing_or_reducing_problems|code": "{{atCode}}",`


  `'MYHQ3'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0021
  1               | at0022
  2               | at0023
  3               | at0024
  4               | at0025  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ3</Code>
  					</QuestionType>
  					<Response>3</Response>
  				</Question>
  ```

  Maps to

  ```json
    "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a3._confident_about_preventing_or_reducing_problems|code": "at0025",
  ```


  #### QuestionType_Code: `'MYHQ4'`:

  ##### JSON target:  
   `"salford_renal_proms_import_report/pam-13:0/point_in_time:0/a4._know_about_medications|code": "{{atCode}}",`


  `'MYHQ4'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0027
  1               | at0028
  2               | at0029
  3               | at0030
  4               | at0031  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ4</Code>
  					</QuestionType>
  					<Response>3</Response>
  				</Question>
  ```

  Maps to

  ```json
    "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a4._know_about_medications|code":
    "at0030",
  ```


  #### QuestionType_Code: `'MYHQ5'`:

  ##### JSON target:  
   ```json
	 "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a5._confident_about_whether_to_go_to_doctor|code": "{{atCode}}",`


  `'MYHQ5'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0033
  1               | at0034
  2               | at0035
  3               | at0036
  4               | at0037  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ5</Code>
  					</QuestionType>
  					<Response>3</Response>
  				</Question>
  ```

  Maps to

  ```json
    "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a5._confident_about_whether_to_go_to_doctor|code": "at0036",
  ```

  #### QuestionType_Code: `'MYHQ6'`:

  ##### JSON target:  
```json
	 "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a6._confident_about_telling_concerns|code": "{{atCode}}",
```

  `'MYHQ6'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0039
  1               | at0040
  2               | at0041
  3               | at0042
  4               | at0043  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ6</Code>
  					</QuestionType>
  					<Response>2</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a6._confident_about_telling_concerns|code": "at0041",
  ```


  #### QuestionType_Code: `'MYHQ7'`:

  ##### JSON target:  
   ```json
	  "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a7._confident_about_home_treatments|code": "{{atCode}}",
```

  `'MYHQ7'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0045
  1               | at0046
  2               | at0047
  3               | at0048
  4               | at0049  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ7</Code>
  					</QuestionType>
  					<Response>2</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a7._confident_about_home_treatments|code": "at0047",
  ```


  #### QuestionType_Code: `'MYHQ8'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a8._understand_health_problems_and_causes|code": {{atCode}}",`


  `'MYHQ8'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0051
  1               | at0052
  2               | at0053
  3               | at0054
  4               | at0055  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ8</Code>
  					</QuestionType>
  					<Response>0</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a8._understand_health_problems_and_causes|code": "at0051",
  ```


  #### QuestionType_Code: `'MYHQ9'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a9._know_about_available_treatments|code": {{atCode}}",`


  `'MYHQ9'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0057
  1               | at0058
  2               | at0059
  3               | at0060
  4               | at0061  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ9</Code>
  					</QuestionType>
  					<Response>0</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a9._know_about_available_treatments|code": "at0057",
  ```


  #### QuestionType_Code: `'MYHQ10'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a10._maintain_lifestyle_changes|code": {{atCode}}",`


  `'MYHQ10'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0063
  1               | at0064
  2               | at0065
  3               | at0066
  4               | at0067  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ10</Code>
  					</QuestionType>
  					<Response>4</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a10._maintain_lifestyle_changes|code": "at0067",
  ```


  #### QuestionType_Code: `'MYHQ11'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a11._know_how_to_prevent_health_problems|code": {{atCode}}",`


  `'MYHQ11'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0069
  1               | at0070
  2               | at0071
  3               | at0072
  4               | at0073  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ11</Code>
  					</QuestionType>
  					<Response>1</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a11._know_how_to_prevent_health_problems|code": "at0070",
  ```


  #### QuestionType_Code: `'MYHQ12'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a12._confident_about_solutions_for_new_health_problems|code": {{atCode}}",`


  `'MYHQ12'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0075
  1               | at0076
  2               | at0077
  3               | at0078
  4               | at0079  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ12</Code>
  					</QuestionType>
  					<Response>1</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a12._confident_about_solutions_for_new_health_problems|code": "at0076",
  ```


  #### QuestionType_Code: `'MYHQ13'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a13._confident_about_maintaining_lifestyle_changes_when_stressed|code": {{atCode}}",`


  `'MYHQ13'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0081
  1               | at0082
  2               | at0083
  3               | at0084
  4               | at0085  

  Example:

  ```xml
  				<Question>
  					<QuestionType>
  						<Code>MYHQ13</Code>
  					</QuestionType>
  					<Response>0</Response>
  				</Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pam-13:0/point_in_time:0/a13._confident_about_maintaining_lifestyle_changes_when_stressed|code": "at0081",
  ```



** `SurveyType/Code = 'PROM'` **
```xml
		<Survey>
			<SurveyTime>2017-03-06T00:00:00</SurveyTime>
			<SurveyType>
				<Code>PROM</Code>
			</SurveyType>
				<Questions>
				 <!-- Snipped for brevity -->
				</Questions>
				<TypeOfTreatment>Pre-dialysis</TypeOfTreatment>
				<HDLocation></HDLocation>
				<Template>Your Health P1 new 01</Template>
			</Survey>
```
##### JSON targets:
  ```json
	"salford_renal_proms_import_report/pos-s_renal:0/history_origin" :"{{SurveyTime}}",
	"salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/_name|value" :"{{TypeOfTreatment}"
	```

PROM (POS-S Renal) does not contain any Score values or levels.

##### Example
  ```json
	"salford_renal_proms_import_report/pos-s_renal:0/history_origin" :"2017-03-06T00:00:00",
	"salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/_name|value" :"Pre-dialysis"
	```

  #### QuestionType_Code: `'YSQ1'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/pain|code": {{atCode}}",`


  `'YSQ1'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0009
  1               | at0010
  2               | at0011
  3               | at0012
  4               | at0013  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ1</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/pain|code": "at0012",
  ```

  #### QuestionType_Code: `'YSQ2'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/shortness_of_breath|code": {{atCode}}",`


  `'YSQ2'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0015
  1               | at0016
  2               | at0017
  3               | at0018
  4               | at0019  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ2</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/shortness_of_breath|code": "at0018",
  ```

  #### QuestionType_Code: `'YSQ3'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/weakness_or_lack_of_energy|code": {{atCode}}",`


  `'YSQ3'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0021
  1               | at0022
  2               | at0023
  3               | at0024
  4               | at0025  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ3</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/weakness_or_lack_of_energy|code": "at0024",
  ```

  #### QuestionType_Code: `'YSQ4'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/nausea|code": {{atCode}}",`


  `'YSQ4'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0027
  1               | at0028
  2               | at0029
  3               | at0030
  4               | at0031  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ4</Code>
            </QuestionType>
            <Response>4</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/nausea|code": "at0031",
  ```

  #### QuestionType_Code: `'YSQ5'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/vomiting|code": {{atCode}}",`


  `'YSQ5'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0033
  1               | at0034
  2               | at0035
  3               | at0036
  4               | at0037  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ5</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/vomiting|code": "at0034",
  ```

  #### QuestionType_Code: `'YSQ6'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/poor_appetite|code": {{atCode}}",`


  `'YSQ6'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0039
  1               | at0040
  2               | at0041
  3               | at0042
  4               | at0043  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ6</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/poor_appetite|code": "at0040",
  ```

  #### QuestionType_Code: `'YSQ7'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/constipation|code": {{atCode}}",`


  `'YSQ7'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0045
  1               | at0046
  2               | at0047
  3               | at0048
  4               | at0049  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ7</Code>
            </QuestionType>
            <Response>4</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/constipation|code": "at0049",
  ```

  #### QuestionType_Code: `'YSQ8'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/mouth_problems|code": {{atCode}}",`


  `'YSQ8'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0051
  1               | at0052
  2               | at0053
  3               | at0054
  4               | at0055  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ8</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/mouth_problems|code": "at0052",
  ```

  #### QuestionType_Code: `'YSQ9'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/drowsiness|code": {{atCode}}",`


  `'YSQ9'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0057
  1               | at0058
  2               | at0059
  3               | at0060
  4               | at0061  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ9</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/drowsiness|code": "at0060",
  ```

  #### QuestionType_Code: `'YS10'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/poor_mobility|code": {{atCode}}",`


  `'YSQ10'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0063
  1               | at0064
  2               | at0065
  3               | at0066
  4               | at0067  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ10</Code>
            </QuestionType>
            <Response>4</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/poor_mobility|code": "at0067",
  ```

  #### QuestionType_Code: `'YS11'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/itching|code": {{atCode}}",`


  `'YSQ11'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0075
  1               | at0076
  2               | at0077
  3               | at0078
  4               | at0079  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ11</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/itching|code": "at0076",
  ```

  #### QuestionType_Code: `'YS12'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/difficulty_sleeping|code": {{atCode}}",`


  `'YSQ12'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0081
  1               | at0082
  2               | at0083
  3               | at0084
  4               | at0085  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ12</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/difficulty_sleeping|code": "at0084",
  ```

  #### QuestionType_Code: `'YS13'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/restless_legs|code": {{atCode}}",`


  `'YSQ13'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0087
  1               | at0088
  2               | at0089
  3               | at0090
  4               | at0091  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ13</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/restless_legs|code": "at0088",
  ```

  #### QuestionType_Code: `'YS14'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/feeling_anxious|code": {{atCode}}",`


  `'YSQ14'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0093
  1               | at0094
  2               | at0095
  3               | at0096
  4               | at0097  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ14</Code>
            </QuestionType>
            <Response>2</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/feeling_anxious|code": "at0095",
  ```


  #### QuestionType_Code: `'YS15'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/feeling_depressed|code": {{atCode}}",`


  `'YSQ15'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0099
  1               | at0100
  2               | at0101
  3               | at0102
  4               | at0103  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ15</Code>
            </QuestionType>
            <Response>2</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/feeling_depressed|code": "at0101",
  ```

  #### QuestionType_Code: `'YS16'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/changes_in_skin|code": {{atCode}}",`


  `'YSQ16'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0105
  1               | at0106
  2               | at0107
  3               | at0108
  4               | at0109  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ16</Code>
            </QuestionType>
            <Response>1</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/changes_in_skin|code": "at0106",
  ```

  #### QuestionType_Code: `'YSQ17'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/diarrhoea|code": {{atCode}}",`


  `'YSQ17'` Response value | Target openEHR `atCode`
  ----------------|--------------
  0               | at0111
  1               | at0112
  2               | at0113
  3               | at0114
  4               | at0115  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YSQ17</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/pos-s_renal:0/point_in_time:0/diarrhoea|code": "at0114",
  ```



** `SurveyType/Code = 'EQ5D'` **
```xml
<Survey>
	<SurveyTime>2017-03-06T00:00:00</SurveyTime>
	<SurveyType>
		<Code>EQ5D</Code>
	</SurveyType>
	<Questions>
	</Questions>
	<TypeOfTreatment>Pre-dialysis</TypeOfTreatment>
	<HDLocation></HDLocation>
	<Template>Your Health P1 new 01</Template>
</Survey>
```

##### JSON targets:
  ```json
	"salford_renal_proms_import_report/eq-5d-5l_questionnaire/history_origin" :"{{SurveyTime}}",
		"salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/_name|value" :"{{TypeOfTreatment}"
	```

	EQ5D (EQ_5D_5L) does not contain any Score values or levels.

##### Example
  ```json
	"salford_renal_proms_import_report/eq-5d-5l_questionnaire/history_origin" :"2017-03-06T00:00:00",
		"salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/_name|value" :"Pre-dialysis"
	```

  #### QuestionType_Code: `'YOHQ1'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/mobility|code": {{atCode}}",`


  `'YOHQ1'` Response value | Target openEHR `atCode`
  ----------------|--------------
  1               | at0005
  2               | at0006
  3               | at0007
  4               | at0008
  5               | at0009  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YOHQ1</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/mobility|code": "at0007",
  ```


  #### QuestionType_Code: `'YOHQ2'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/self_care|code": {{atCode}}",`


  `'YOHQ2'` Response value | Target openEHR `atCode`
  ----------------|--------------
  1               | at0011
  2               | at0012
  3               | at0013
  4               | at0014
  5               | at0015  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YOHQ2</Code>
            </QuestionType>
            <Response>5</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/self_care|code": "at0015",
  ```


  #### QuestionType_Code: `'YOHQ3'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/usual_activities|code": {{atCode}}",`


  `'YOHQ3'` Response value | Target openEHR `atCode`
  ----------------|--------------
  1               | at0017
  2               | at0018
  3               | at0019
  4               | at0020
  5               | at0021  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YOHQ3</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/usual_activities|code": "at0019",
  ```


  #### QuestionType_Code: `'YOHQ4'`:

  ##### JSON target:  
   ```json
	 "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/pain_or_discomfort|code": "{{atCode}}",
	 ```


  `'YOHQ4'` Response value | Target openEHR `atCode`
  ----------------|--------------
  1               | at0023
  2               | at0024
  3               | at0025
  4               | at0026
  5               | at0027  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YOHQ4</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/pain_or_discomfort|code": "at0025",
  ```


  #### QuestionType_Code: `'YOHQ5'`:

  ##### JSON target:  
   ` "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/anxiety_or_depression|code": {{atCode}}",`


  `'YOHQ5'` Response value | Target openEHR `atCode`
  ----------------|--------------
  1               | at0031
  2               | at0032
  3               | at0033
  4               | at0034
  5               | at0035  

  Example:

  ```xml
          <Question>
            <QuestionType>
              <Code>YOHQ5</Code>
            </QuestionType>
            <Response>3</Response>
          </Question>
  ```

  Maps to

  ```json
      "salford_renal_proms_import_report/eq-5d-5l_questionnaire:0/point_in_time:0/anxiety_or_depression|code": "at0033",
  ```
