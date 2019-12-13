﻿---
title: Evaluate ServiceDefinition Interaction
keywords: servicedefinition, rest,
tags: [rest,fhir,api]
sidebar: ctp_rest_sidebar
permalink: api_post_service_definition.html
summary: Evaluate ServiceDefinition interaction
---

{% include custom/search.warnbanner.html %}
<!--
{% include custom/fhir.referencemin.html resource="" userlink="" page="" fhirname="Service Definition" fhirlink="[Service Definition](http://hl7.org/fhir/stu3/servicedefinition.html)" content="User Stories" userlink="" %}
-->

## Evaluate ServiceDefinition Interaction ##
This is a [FHIR operation](https://www.hl7.org/fhir/stu3/operations.html) performed by the Encounter Management System (EMS). 
It is an [evaluate operation](https://www.hl7.org/fhir/stu3/servicedefinition-operations.html#evaluate) performed against the [Service Definition](http://hl7.org/fhir/stu3/servicedefinition.html) resource to request clinical decision support guidance from a selected Clinical Decision Support System (CDSS).

### Trigger for Evaluate ServiceDefinition Interaction ###  
The `ServiceDefinition.trigger` element is of datatype [TriggerDefinition](https://www.hl7.org/fhir/stu3/metadatatypes.html#TriggerDefinition) and this structure defines when a knowledge artifact, in this case a `ServiceDefinition`, is expected to be evaluated.  
Within the CDS implementation, the Data Event trigger type has been chosen. This means that the EMS's evaluation of a `ServiceDefinition` will be triggered in response to a data-related activity within an implementation, for example by an addition or an update of a record such as a `QuestionnaireResponse` resource.  
The triggering data of the event is described in the `trigger.eventData` element of the `ServiceDefinition` and this is populated by the CDSS.     

## Request Headers ##
The following HTTP request headers are supported for this interaction:  


| Header               | Value |Conformance |
|----------------------|-------|-------|
| `Accept`      | The `Accept` header indicates the format of the response the client is able to understand, this will be one of the following `application/fhir+json` or `application/fhir+xml`. See the RESTful API [Content types](api_general_guidance.html#content-types) section. | MAY |
| `Authorization`      | The `Authorization` header MUST carry a base64url encoded JSON web token. See the RESTful API [Security](api_security.html) section. | MUST |  

## POST ServiceDefinition ##

The `ServiceDefinition.$evaluate` operation is performed by an HTTP POST command as shown:  

<div markdown="span" class="alert alert-success" role="alert">
POST [base]/ServiceDefinition/[id]/$evaluate</div>  

## Parameters ##
The `ServiceDefinition.$evaluate` operation has a number of parameters and the EMS will select appropriate IN parameters to include in the operation. The [Parameters](https://fhir.nhs.uk/STU3/StructureDefinition/CDS-EvaluateServiceDefinition-Parameters-1) profile should be used to carry the parameter values.
The CDSS will return a `GuidanceResponse` resource as the OUT parameter of the operation.  

### IN Parameters ###

<table style="min-width:100%;width:100%">
<tr>
    <th style="width:10%;">Name</th>
    <th style="width:5%;">Cardinality</th>
    <th style="width:10%;">Type</th>
      <th style="width:40%;">FHIR Documentation</th>
   <th style="width:35%;">CDS Implementation Guidance</th>
</tr>

<tr>
  <td><code class="highlighter-rouge">requestId</code></td>
    <td><code class="highlighter-rouge">0..1</code></td>
    <td>id</td>
    <td>An optional client-provided identifier to track the request.</td>
<td>This MUST be populated.<br/>
Each invocation of the $evaluate method MUST use a unique requestId<br/>
The requestId MUST be locally unique</td>
</tr>
<tr>
   <td><code class="highlighter-rouge">evaluateAtDateTime</code></td>
   <td><code class="highlighter-rouge">0..1</code></td>
    <td>dateTime</td>
    <td>
	An optional date and time specifying that the evaluation should be performed as though it was the given date and time. The most direct implication of this is that references to "Now" within the evaluation logic of the module should result in this value. In addition, wherever possible, the data accessed by the module should appear as though it was accessed at this time. The evaluateAtDateTime value may be any time in the past or future, enabling both retrospective and prospective scenarios. If no value is provided, the date and time of the request is assumed.
	</td>
 <td></td>
</tr>
<tr>
    <td><code class="highlighter-rouge">inputParameters</code></td>
    <td><code class="highlighter-rouge">0..1</code></td>
    <td>Parameters</td>
    <td></td>
</tr>
<tr>
    <td><code class="highlighter-rouge">inputData</code></td>
     <td><code class="highlighter-rouge">0..1</code></td>
    <td>Any</td>
    <td>The input data for the request. These data are defined by the data requirements of the module and typically provide patient-dependent information.</td>
   <td>The <a href="api_post_service_definition.html#inputdata-element">inputData element</a> MUST be populated with FHIR resources detailing the current state of the triage journey as follows:  
<ul>  
<li>All assertions current for this Encounter (typically Observation resources). This includes all GuidanceResponse outputParameters supplied by any CDSS. Any relevant information taken from other (external) systems SHOULD be included.</li> 

<li>Any QuestionnaireResponse(s) available from the user. Note that this MAY include QuestionnaireResponse(s) which have been amended. These MUST be addressed by the CDSS and the assertions updated.
The CDSS MUST filter the supplied inputData and disregard any information not relevant for the ServiceDefinition.
The EMS MUST NOT send duplicate items.</li>
</ul>
</td>
</tr> 
<tr>
  <td><code class="highlighter-rouge">patient</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
    <td>Reference<br>(Patient)</td>
    <td>The patient in context, if any.</td>
	<td></td>
 </tr>
<tr>
	<td><code class="highlighter-rouge">encounter</code></td>
    <td><code class="highlighter-rouge">0..1</code></td>
    <td>Reference<br>(Encounter)</td>
    <td>The encounter in context, if any.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">initiatingOrganization</code></td>
        <td><code class="highlighter-rouge">0..1</code></td>
    <td>Reference<br>(Organization)</td>
    <td>The organisation initiating the request.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">initiatingPerson</code></td>
    <td><code class="highlighter-rouge">0..1</code></td>
    <td>Reference<br>(Patient |<br>Practitioner |<br>RelatedPerson)</td>
    <td>The person initiating the request.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">userType</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
    <td>CodeableConcept</td>
    <td>The type of user initiating the request, e.g. patient, healthcare provider, or specific type of healthcare provider (physician, nurse, etc.).</td>
	<td></td>
 </tr>
<tr>
    <td><code class="highlighter-rouge">userLanguage</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
    <td>CodeableConcept</td>
    <td>Preferred language of the person using the system.</td>
	<td></td>
 </tr>
<tr>
   <td><code class="highlighter-rouge">userTaskContext</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
     <td>CodeableConcept</td>
    <td>The task the system user is performing, e.g. laboratory results review, medication list review, etc. This information can be used to tailor decision support outputs, such as recommended information resources.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">receivingOrganization</code></td>
        <td><code class="highlighter-rouge">0..1</code></td>
	<td>Reference<br>(Organization)</td>
    <td>The organization that will receive the response.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">receivingPerson</code></td>
        <td><code class="highlighter-rouge">0..1</code></td>
   <td>Reference<br>(Patient |<br>Practitioner |<br>RelatedPerson)</td>
    <td>The person in the receiving organization who will receive the response.</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">recipientType</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
    <td>CodeableConcept</td>
    <td>The type of individual that will consume the response content. This may be different from the requesting user type (e.g. if a clinician is getting disease management guidance for provision to a patient). E.g. patient, healthcare provider or specific type of healthcare provider (physician, nurse, etc.).</td>
	<td></td>
 </tr>
<tr>
   <td><code class="highlighter-rouge">recipientLanguage</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
     <td>CodeableConcept</td>
    <td>Preferred language of the person that will consume the content.</td>
	<td></td>
  </tr>
<tr>
   <td><code class="highlighter-rouge">setting</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
     <td>CodeableConcept</td>
    <td>The current setting of the request (inpatient, outpatient, etc).</td>
	<td></td>
  </tr>
<tr>
    <td><code class="highlighter-rouge">settingContext</code></td>
      <td><code class="highlighter-rouge">0..1</code></td>
    <td>CodeableConcept</td>
    <td>Additional detail about the setting of the request, if any.</td>
	<td>This MUST NOT be populated.</td>
</tr>
</table>


### OUT Parameters ###

<table style="min-width:100%;width:100%">
<tr>
    <th style="width:25%;">Name</th>
    <th style="width:15%;">Cardinality</th>
    <th style="width:20%;">Type</th>
      <th style="width:40%;">Documentation</th>
</tr>
<tr>
    <td><code class="highlighter-rouge">return</code></td>
    <td><code class="highlighter-rouge">1..1</code></td>
    <td>GuidanceResponse</td>
    <td>The result of the request as a GuidanceResponse resource.  
	Note: as this the only out parameter, it is a resource, and it has the name 'return', the result of this operation is returned directly as a resource.</td>
 </tr>
</table>

## ServiceDefinition $evaluate parameters of note ##
Parameters passed in the `ServiceDefinition.$evaluate` operation of particular significance to implementers are noted below:  

### encounter element ###
This element carries reference to the encounter currently in context for the triage. It is the responsibility of the EMS to identify the correct encounter in each `$evaluate` interaction with the CDSS to reduce the risk of inappropriate triage.

### inputData element ###
This element carries the triage journey data for the decision. The EMS will populate this element with all assertions collected in a particular triage journey and any other assertions considered to be of relevance.
The CDSS is obliged to consider anything carried in the `ServiceDefinition.dataRequirement` element, but can ignore resources posted in inputData which are not in the `dataRequirement` element.
In particular, where there are ‘branching’ or bundled `ServiceDefinitions` within a single journey, the last `ServiceDefinition` will have all assertions passed in inputData from prior ServiceDefinitions.
This may or may not affect the population of the result element in `GuidanceResponse` created by the CDSS.

### patient element ###

This element carries reference to the `Patient` currently undergoing triage. It is the responsibility of the EMS to identify the correct patient in each `$evaluate` interaction with the CDSS to reduce the risk of inappropriate triage.

### userType element ###
The `userType` element carries the type of user initiating the request to the CDSS. It MUST be populated by the EMS as it has an important relationship with two other optional parameters, the `initiatingPerson` and the `receivingPerson`.  
The relationship between these elements and the concepts they carry are outlined below:

<table style="min-width:100%;width:100%">
<tr>
    <th style="width:25%;">FHIR element</th>
    <td style="width:25%;">userType</td>
    <td style="width:25%;">initiatingPerson</td>
      <td style="width:25%;">receivingPerson</td>
</tr>
<tr>
    <th>Business concept</th>
      <td>Type (role) of the EMS user</td>
    <td>User of the EMS</td>
    <td>Person acting next (receiving the result)</td>
 </tr>
 <tr>
    <th>Scenarios</th>
      <td></td>
    <td></td>
    <td></td>
 </tr>
  <tr>
    <td>Patient using EMS on their own behalf</td>
      <td>Patient</td>
    <td>Reference(Patient)</td>
    <td>Reference(Patient)</td>
 </tr>
   <tr>
    <td>Person related to patient using EMS on behalf of the patient</td>
      <td>Related person</td>
    <td>Reference(RelatedPerson)</td>
      <td>Reference(RelatedPerson)</td>
 </tr>
    <tr>
    <td>Patient speaking to practitioner using EMS on behalf of the patient</td>
      <td>Practitioner type</td>
    <td>Reference(Practitioner)</td>
      <td>Reference(Patient)</td>
 </tr>
 <tr>
    <td>Person related to patient speaking to practitioner using EMS on behalf of the patient</td>
      <td>Practitioner type</td>
    <td>Reference(Practitioner)</td>
      <td>Reference(RelatedPerson)</td>
 </tr>
</table>





## Response from CDSS ##

### Success ###

* MUST return a <code class="highlighter-rouge">200</code> **OK** HTTP status code on successful execution of the operation.
* MUST return a <code class="highlighter-rouge">GuidanceResponse</code> resource.  

<a href="api_return_guidance_response.html">View further information about the result of the triage journey</a>.  

## Time out ##  
It is recommended that the EMS sets a time out limit on the response back from the CDSS, appropriate to an interactive process (e.g. around 1000 milliseconds).  
If the CDSS does not respond within the time out period, then it is recommended that the EMS retry the `$evaluate` operation. This is to allow for intermittent network errors.  
After a limited number of retries (e.g. 3-5) the EMS may assume that the CDSS is unavailable and should respond appropriately to the user.

<!--
#### GuidanceResponse Statuses ####
The returned [GuidanceResponse resource](api_guidance_response.html) MUST carry an appropriate code in its <code class="highlighter-rouge">status</code> element.    
This is a trigger for the EMS and the relevant codes are as follows:-  

<table style="min-width:100%;width:100%">
<tr>
    <th style="width:25%;">Code</th>
    <th style="width:25%;">Display</th>
    <th style="width:50%;">Definition</th>
</tr>
<tr>
    <td><code class="highlighter-rouge">success</code></td>
      <td>Success</td>
    <td>The request was processed successfully</td>
 </tr>
<tr>
    <td><code class="highlighter-rouge">data-requested</code></td>
      <td>Data Requested</td>
    <td>The request was processed successfully, but more data may result in a more complete evaluation</td>
 </tr>
<tr>
    <td><code class="highlighter-rouge">data-required</code></td>
      <td>Data Required</td>
    <td>The request was processed, but more data is required to complete the evaluation</td>
 </tr>
</table>
-->

<!--
### Failure ###
The following errors can be triggered when performing this operation:  

More errors are likely to be needed once we are clear on which search parameters are defined

* [Invalid parameter](api_general_guidance.html#parameters)
* [No record found](api_general_guidance.html#resource-not-found---servicedefinition)
-->
<!--
## Example Scenario ##
 Placeholder -->













