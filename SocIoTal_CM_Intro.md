#SOCIOTAL CONTEXT MANAGER INTRODUCTION

<span id="_Ref420668053" class="anchor"><span id="_Toc421274687" class="anchor"></span></span>Description
=========================================================================================================

SocIoTal (centralized) Context Manager acts as one of the core blocks of integration within SocIoTal Integrated Platform. As shown in Figure 1, it provides the SocIoTal entities directory and storage for its current context information plus a complete NGSI9 and NGSI10 API rest interfaces, detailed (and updated) in [*SocIoTal Wiki*](https://github.com/sociotal/SOCIOTAL/wiki/SocIoTal-Context-Manager) and extended with user functionalities in order to provide access and manage all this set of information.

| <img src="https://github.com/IEMaestro/Context-Manager/blob/master/documentation/CM_Arch.png" width="538" height="367" />                                                                                             |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.  <span id="_Ref420597001" class="anchor"><span id="_Toc421113559" class="anchor"></span></span>SocIoTal Framework. Centralized Context Manager architecture |

Besides managing all context information related to entities, it also provides support to SocIoTal’s Trust & Reputation mechanisms and SocIoTal’s bubbles creation, keeping and supplying information related to scores and attributes. SocIoTal Context Manager is also integrated with the SocIoTal Security Framework by evaluating the Capability-Token and supports communities’ management through the corresponding Community Token.

Capability-Token support
------------------------

Access to SocIoTal Context Manager (Version 3 deployed) can be configured to be secured by the SocIoTal Security Framework (see configuration file in applications folder). Integration between these two SocIoTal components is performed through the Capability-Token and the Capability Evaluator (see Security section in github). The Capability-Token, obtained from the Capability Manager, contains all the required credentials of the requestor identity plus the action requested. As shown in [*SocIoTal WiKi*](https://github.com/sociotal/SOCIOTAL/wiki/SocIoTal-Context-Manager), to call SocIoTal Context Manager V3 methods (either, NGSI9, NGSI10 or EXTENDED interfaces) a “Capability-Token” header is required. This header will contain the corresponding capability token to be checked by the capability evaluator, through the libraries included in the Context Manager V3 deployment.

In order to authenticate the identity of the requestor included in the capability token, to alternative ways have been integrated in SocIoTal Context Manager V3:

-   Through User Certificate: using the PKI-based authentication, the user will attach the “Signature” header, including the corresponding signature linked to its certificate. This signature will be captured and passed (by the SocIoTal CM) to the capability evaluator library that will execute the authentication process.

-   Through IdM Authentication-Token: the SocIoTal IdM, using its KeyRock core, will provide an authentication token, linked to the user identity. This token will be provided to the SocIoTal CM in the Auth-Token header, and the user requestor id in the Client-Id header. These two parameters will be later used by the capability evaluator library to authenticate, against the IdM KeyRock, the user identity.

Once the capability evaluator library checks both, the supplied capability token plus the user authentication, will authorize or deny (including the corresponding rejection message) the action requested. Then, the SocIoTal Context Manager will execute the action (if allowed) or return the rejection message to the requestor application (when action is denied).

Community-Token support
-----------------------

A SocIoTal Community builds a closed environment where only registered users and entities can share information, representing a group of users and resources with a common objective or inquisitiveness. The SocIoTal component that manages communities and users/entities memberships is the Communities Manager.

The integration with communities’ management is done through the corresponding Community-Token. Supposing an already registered user, also member of a community, this user can request a community-token that identifies (and authenticates) this user as member of this community to the SocIoTal Communities Manager. The methods to achieve this are detailed in the [*SocIoTal WiKi*](https://github.com/sociotal/SOCIOTAL/wiki/SocIoTal-Communities-Manager). The retrieved UUID (Universally Unique Identifier) of the generated Community-Token can be added now, as the “Community-Token” header, to every request (update, discover, create or subscribe) addressed to the Context Manager. SocIoTal Context Manager supports two different operation modes related to Communities:

-   With no “Community-Token” header: the request will be addressed to a default community within a default domain. This is, all resources are assumed to belong to the same community. There will be no users/roles (from the point of view of communities) filtering and only SocIoTal security framework policies and restrictions (through the required Capability-Token) will apply. This could be useful when an instance of SocIoTal is going to be used for a single and well defined purpose, such as the platform for a particular application.

-   When “Community-Token” is attached to the request. This token will be validated by the Communities Manager and info related to it will be retrieved to the Context Broker (as mentioned in the section above for API endpoints). The Context Broker here will apply the corresponding communities’ restrictions/actions required, depending on the requested method and the info obtained from the validated Community-Token.

SocIoTal Context Manager Implementation
---------------------------------------

First implementation of SocIoTal Context Manager has been built over selected FIWARE platform, after an analysis of existing candidate suitable platforms and requirements support.

| <img src="https://github.com/IEMaestro/Context-Manager/blob/master/documentation/CM_Implementation_Diagram.png" width="780" height="500" />                                                                                       |
|---------------------------------------------------------------------------------------------------------------------------------------------------|
| 2.  <span id="_Ref420667127" class="anchor"><span id="_Toc421113567" class="anchor"></span></span>Current SocIoTal Context Manager implementation |

As shown in Figure 2, FIWARE architecture provides Orion Context Broker enabler, a NGSI9/10 compatible broker that also supports MongoDB storage to implement SocIoTal CM Context Information Storage and Entities’ Directory.

SocIoTal CM has been built to be adapted to any other context broker supporting NGSI9/10 communication interfaces by developing only the corresponding adaptation layer. This way, all functionalities and APIs provided by SocIoTal CM to users and developers should remain the same, regardless of the core platform or the context broker used. Current working SocIoTal CM version is provided with the FIWARE and Orion adaptor embedded.

The **Context Communication Module** provides three main functionalities:

-   **NGSI9 API RESTFul interface**: it implements and provides support for NGSI9 operations, oriented to register, update and discover Context Entities, their attributes and availability. It is composed by the following methods:

    -   registerContext (registerContextRequest/Response)

    -   discoverContextAvailablitiy (discoverContextAvailabilityRequest/Response)

    -   subscribeContextAvailability (subscribeContextAvailabilityRequest/Response)

    -   updateContextAvailabilitySubscription (updateContextAvailabilitySubscriptionRequest/Response))

    -   unsubscribeContextAvailability (unsubscribeContextAvailabilityRequest/Response)

    -   notifyContextAvailability (notification to be sent to subscribed apps)

-   **NGSI10 API RESTFul interface**: it implements and provides support for NGSI10 operations, oriented to register, update and discover Context Information, providing the attributes’ values and defining the associated metadata. It is composed by the following methods:

    -   queryContext (queryContextRequest/Response)

    -   updateContext (updateContextRequest/Response)

    -   subscribeContext (subscribeContextRequest/Response)

    -   unsubscribeContext(unsubscribeContextRequest/Response)

    -   updateContextSubscription (updateContextSubscriptionRequest/Response))

    -   notifyContext (notification to be sent to subscribed apps)

-   **SocIoTal PEP:** as a Policy Enforcement Point, it will capture the capability token that comes with each request and check it against the Capability Manager, within the SocIoTal Security Framework, in order to accept or reject the method call.

The **Context Information Management** module reads and process the context requests and intercepts all the context responses, processing the payloads contents to implement filtering functionalities (related to communities and bubbles membership, specific attribute contents and subscription capabilities) and other sort of features provided by SocIoTal. This module allows SocIoTal context management and interconnection with other SocIoTal components (such Trust & Reputation enabler, F2F, Security Framework) to build the SocIoTal complete platform and set of tools.

Once each context request content has been processed, the **OMA NGSI Adapter** element converts it on a pure NGSI9/10 request (besides the NGSI9/10 API, SocIoTal CM is able to implement further functionalities, extending its context features and offering developers more efficient methods) to be sent to the core platform context broker (FIWARE Orion CB in this implementation). Before this final request reaches the Context Broker, the corresponding **platform adaptor** reformat the NGSI9/10 request according to the selected core NGSI9/10 specifications. As already mentioned, current SocIoTal CM implementation supports FIWARE Orion Context Broker integration. This implementation also uses its own instance of Orion, in order to initially protect all data shared within SocIoTal trials and pilots but SocIoTal CM release version would be easily linked to any other available Orion instance, either provided by FILAB or deployed in a specific project scope/developers environment.

Context Manager performance
===========================

A cloud instance of the SocIoTal CM and its context communication API (the standard way to access SocIoTal CM features) is up and listening in ****platform.sociotal.eu:3500**** (http) and ****platform.sociotal.eu:8443**** (https), the cloud instance of SocIoTal Integrated Platform, providing http support for the functionalities mentioned above and described in [*SocIoTal WiKi*](https://github.com/sociotal/SOCIOTAL/wiki/SocIoTal-Context-Manager). This section, as performance examples, will show real executions focused on the current provided discovery functionalities. The following are examples, so, IDs, IP addresses and specific data might be changed during updates and upgrades. Please, take them as examples. Updated information about methods and payloads will be provided through the [*SocIoTal WiKi*](https://github.com/sociotal/SOCIOTAL/wiki/SocIoTal-Context-Manager). Examples here use http access but could be also executed over https protocol.

registerContext and updateContext
---------------------------------

In order to execute all the tests on discovering context and accessing context information, first, several entities were created, belonging to the UC Weather Station scenario. These entities are Arduino and Raspberry-Pi based and support temperature and humidity sensors. As part of the IoT Meetups and the developers sessions planned within SocIoTal engagement activities, these devices implements either, a first version of the On Device CM that captures sensor data, conforms the corresponding context information and updates it on the SocIoTal centralized CM or the link to the available gateway to capture device sensor information and update it on the CM. These resources are called *“Santander Weather Stations 00X”*, defining their associated entity IDs as

“***SocIoTal:SAN:WeatherStation:Dev\_00X***” (from now on named in the text WS:Dev\_00X)

Once created, and before start producing context information, they were registered including their attribute names and availability.

| SocIoTal SERVER                                                                               | platform.sociotal.eu:3500                          |
|-----------------------------------------------------------------------------------------------|----------------------------------------------------|
| POST                                                                                          | /SocIoTal\_CM\_REST\_V3/NGSI9\_API/registerContext |
| HEADERS                                                                                       |                                                    |
|                                                                                               | Content-type                                       |
|                                                                                               | Accept                                             |
| If required                                                                                   | Capability-token                                   |
| If required                                                                                   | Community-Token                                    |
| *If required*                                                                                 | *X-Auth-Token*                                     |
| registerContextRequest                                                                        |
| {"contextRegistrations": \[{                                                                  
                                                                                                
 "entities": \[{                                                                                
                                                                                                
 "id": "SocIoTal:SAN:WeatherStation:Dev\_001",                                                  
                                                                                                
 "type": "urn:x-org:sociotal:resource:device",                                                  
                                                                                                
 "isPattern": "false"                                                                           
                                                                                                
 }\],                                                                                           
                                                                                                
 "attributes": \[{                                                                              
                                                                                                
 "name": "AmbientTemperature",                                                                  
                                                                                                
 "isDomain": "false",                                                                           
                                                                                                
 "type": "http://sensorml.com/ont/swe/property/AmbientTemperature"                              
                                                                                                
 },                                                                                             
                                                                                                
 {                                                                                              
                                                                                                
 "name": "HumidityValue",                                                                       
                                                                                                
 "isDomain": "false",                                                                           
                                                                                                
 "type": "http://sensorml.com/ont/swe/property/HumidityValue"                                   
                                                                                                
 },                                                                                             
                                                                                                
 {                                                                                              
                                                                                                
 "name": "Location",                                                                            
                                                                                                
 "isDomain": "false",                                                                           
                                                                                                
 "type": "http://sensorml.com/ont/swe/property/Location"                                        
                                                                                                
 }\],                                                                                           
                                                                                                
 "providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API"  
                                                                                                
 }\],                                                                                           
                                                                                                
 "duration": "P1M"                                                                              
                                                                                                
 }                                                                                              |

The method call shown above describes the context, as a set of attributes within the payload. It includes a list of ***contextRegistration*** elements, each one with the following information:

-   **entities**: lists all entities to be registered within the call. Here has only been registered *SocIoTal:SAN:WeatherStation:Dev\_001*, but one payload can be used to register several entities. Besides its ID, it has been also specified the type of entity (*urn:x-org:sociotal:resource:device*). Initially, there are no restrictions nor templates when defining both, the type and the ID, but creating it wisely will help on resources classification and discovery procedures.

-   **attributes**: a list of attributes to register for the entities. In our case, they are the “*AmbientTemperature*”, “*HumidityValue*” and “*Location*” attributes, including a name and a type according to SocIoTal Context Model.

-   **providing application** (or Context Provider): the URL that represents the actual context information for the entities and attributes being registered. In this case, related context information (attribute values) is stored and provided by the SocIoTal CM itself \[*http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API*\]

-   **duration**: sets the duration of the registration so after that time has passed it can be considered as expired (however, duration can be extended). ISO 8601 standard is used for duration format, for example "P1M" means "one month".

The response payload returned shows that everything resulted as expected, returning the registration ID (to be further used to modify the entity) and confirming the requested duration.

| registerContextResponse                      |
|----------------------------------------------|
| {                                            
                                               
 "duration": "P1M",                            
                                               
 "registrationId": "549304a81860a3e2039f5ae4"  
                                               
 }                                             |

This way, the WS:Dev\_001 was registered into the SocIoTal Entities’ Directory. After this, the entity can start generating context information and can also be discovered by other users/entities.

updateContext provided method includes 3 main functionalities, pointed through the “updateaction” element:

-   APPEND creates/modifies the template for a new context entity to provide its context information, detailing its attributes and associated metadata. It complements registerContext method specifying the context information.

-   UPDATE replaces existing attributes values (and metadata) of a given entity (or context element) with the provided values

-   DELETE removes an existing context information set, related to an entity. It will not remove the specified entity from the Entities’ Directory

The following updateContext Append call defines the context information template for WS:Dev\_001 and provides also the corresponding initial attribute values

| SocIoTal SERVER                                                                                                                 | platform.sociotal.eu:3500                         |
|---------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|
| POST                                                                                                                            | /SocIoTal\_CM\_REST\_V3/NGSI10\_API/updateContext |
| HEADERS                                                                                                                         |                                                   |
|                                                                                                                                 | Content-type                                      |
|                                                                                                                                 | Accept                                            |
| If required                                                                                                                     | Capability-token                                  |
| If required                                                                                                                     | Community-Token                                   |
| *If required*                                                                                                                   | *X-Auth-Token*                                    |
| updateContextRequest                                                                                                            |
| {                                                                                                                               
                                                                                                                                  
 "contextElements": \[{                                                                                                           
                                                                                                                                  
 "type": " urn:x-org:sociotal:resource:device ",                                                                                  
                                                                                                                                  
 "isPattern": "false",                                                                                                            
                                                                                                                                  
 "id": "SocIoTal:SAN:WeatherStation:Dev\_001",                                                                                    
                                                                                                                                  
 "attributes": \[{                                                                                                                
                                                                                                                                  
 "name": "AmbientTemperature",                                                                                                    
                                                                                                                                  
 "value": "25.44",                                                                                                                
                                                                                                                                  
 "type": "[*http://sensorml.com/ont/swe/property/AmbientTemperature*](http://sensorml.com/ont/swe/property/AmbientTemperature)",  
                                                                                                                                  
 "metadatas": \[{                                                                                                                 
                                                                                                                                  
 "name": "DateTimeStamp",                                                                                                         
                                                                                                                                  
 "value": "20141030T113343Z",                                                                                                     
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/DateTimeStamp"                                                                     
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "Unit",                                                                                                                  
                                                                                                                                  
 "value": "celsius",                                                                                                              
                                                                                                                                  
 "type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"                                                                              
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "accuracy",                                                                                                              
                                                                                                                                  
 "value": "0,5",                                                                                                                  
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"                                                     
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "DataDescription",                                                                                                       
                                                                                                                                  
 "value": "float",                                                                                                                
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/DataDescription"                                                                   
                                                                                                                                  
 }\]                                                                                                                              
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "HumidityValue",                                                                                                         
                                                                                                                                  
 "value": "59",                                                                                                                   
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/HumidityValue",                                                                    
                                                                                                                                  
 "metadatas": \[{                                                                                                                 
                                                                                                                                  
 "name": "DateTimeStamp",                                                                                                         
                                                                                                                                  
 "value": "20141030T113343Z",                                                                                                     
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/DateTimeStamp"                                                                     
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "Unit",                                                                                                                  
                                                                                                                                  
 "value": "percentage",                                                                                                           
                                                                                                                                  
 "type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"                                                                              
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "accuracy",                                                                                                              
                                                                                                                                  
 "value": "1",                                                                                                                    
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"                                                     
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "DataDescription",                                                                                                       
                                                                                                                                  
 "value": "integer",                                                                                                              
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/DataDescription"                                                                   
                                                                                                                                  
 }\]                                                                                                                              
                                                                                                                                  
 },                                                                                                                               
                                                                                                                                  
 {                                                                                                                                
                                                                                                                                  
 "name": "Location",                                                                                                              
                                                                                                                                  
 "value": "43.472057, -3.800156",                                                                                                 
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/Location",                                                                         
                                                                                                                                  
 "metadatas": \[{                                                                                                                 
                                                                                                                                  
 "name": "WorldGeographicReferenceSystem",                                                                                        
                                                                                                                                  
 "value": "WSG84",                                                                                                                
                                                                                                                                  
 "type": "http://sensorml.com/ont/swe/property/WorldGeographicReferenceSystem"                                                    
                                                                                                                                  
 }\]                                                                                                                              
                                                                                                                                  
 }\],                                                                                                                             
                                                                                                                                  
 }\],                                                                                                                             
                                                                                                                                  
 "updateAction": "APPEND"                                                                                                         
                                                                                                                                  
 }                                                                                                                                |

The below updateContext This update call is used every time a new context information is available from the WS:Dev\_001. It will put the last provided value and remove the previous one on the attributes and the metadata specified in the payload. Those attributes not included in the payload will keep the stored values. The presented call provides update context information related to AmbientTemperature.

| SocIoTal SERVER                                                              | platform.sociotal.eu:3500                         |
|------------------------------------------------------------------------------|---------------------------------------------------|
| POST                                                                         | /SocIoTal\_CM\_REST\_V3/NGSI10\_API/updateContext |
| HEADERS                                                                      |                                                   |
|                                                                              | Content-type                                      |
|                                                                              | Accept                                            |
| If required                                                                  | Capability-token                                  |
| If required                                                                  | Community-Token                                   |
| *If required*                                                                | *X-Auth-Token*                                    |
| updateContextRequest                                                         |
| {                                                                            
                                                                               
 "contextElements": \[{                                                        
                                                                               
 "id": "SocIoTal:SAN:WeatherStation:Dev\_001",                                 
                                                                               
 "type": "urn:x-org:sociotal:resource:device",                                 
                                                                               
 "isPattern": "false",                                                         
                                                                               
 "attributes": \[{                                                             
                                                                               
 "name": "AmbientTemperature",                                                 
                                                                               
 "value": "25.44",                                                             
                                                                               
 "type": "<http://sensorml.com/ont/swe/property/AmbientTemperature>",          
                                                                               
 "metadatas": \[{                                                              
                                                                               
 "name": "DateTimeStamp",                                                      
                                                                               
 "value": "20141030T113343Z",                                                  
                                                                               
 "type": "http://sensorml.com/ont/swe/property/DateTimeStamp"                  
                                                                               
 },                                                                            
                                                                               
 {                                                                             
                                                                               
 "name": "Unit",                                                               
                                                                               
 "value": "celsius",                                                           
                                                                               
 "type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"                           
                                                                               
 },                                                                            
                                                                               
 {                                                                             
                                                                               
 "name": "accuracy",                                                           
                                                                               
 "value": "0,5",                                                               
                                                                               
 "type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"  
                                                                               
 },                                                                            
                                                                               
 {                                                                             
                                                                               
 "name": "DataDescription",                                                    
                                                                               
 "value": "float",                                                             
                                                                               
 "type": "http://sensorml.com/ont/swe/property/DataDescription"                
                                                                               
 }\]                                                                           
                                                                               
 }\]                                                                           
                                                                               
 }\],                                                                          
                                                                               
 "updateAction": "UPDATE"                                                      
                                                                               
 }                                                                             |

Further possibilities and operations related to entities registration and context updates can be found on SocIoTal GitHub.

### discoverContextAvailability

Once the context has been uploaded in the SocIoTal CM, can be now discovered. The following discoveryContextAvailability call will discover all SocIoTal entities providing AmbientTemperature context information.

| SocIoTal SERVER                                                                            | platform.sociotal.eu:3500                                      |
|--------------------------------------------------------------------------------------------|----------------------------------------------------------------|
| POST                                                                                       | /SocIoTal\_CM\_REST\_V3/NGSI9\_API/discoverContextAvailability |
| HEADERS                                                                                    |                                                                |
|                                                                                            | Content-type                                                   |
|                                                                                            | Accept                                                         |
| If required                                                                                | Capability-token                                               |
| If required                                                                                | Community-Token                                                |
| *If required*                                                                              | *X-Auth-Token*                                                 |
| discoverContextAvailabilityRequest                                                         |
| {                                                                                          
                                                                                             
 "entities": \[{                                                                             
                                                                                             
 "type": "",                                                                                 
                                                                                             
 "isPattern": "true",                                                                        
                                                                                             
 "id": "SocIoTal.\*" *//Pattern to search for all entities with ID starting with SocIoTal:*  
                                                                                             
 }\],                                                                                        
                                                                                             
 "attributes": \["AmbientTemperature"\]                                                      
                                                                                             
 }                                                                                           |

The response (below) includes all available entities, with their corresponding entity IDs and the available context specified (this time, AmbientTemperatue)

discoverContextAvailabilityResponse

{

"contextRegistrationResponses": \[

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[

{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

}

\],

"entities": \[

{

**"id": "SocIoTal:SAN:WeatherStation:Dev\_001",**

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

\]

}

},

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[

{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

}

\],

"entities": \[

{

**"id": "SocIoTal:SAN:WeatherStation:Dev\_002",**

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

\]

}

},

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[

{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

}

\],

"entities": \[

{

**"id": "SocIoTal:CRS4:WeatherStation:Dev\_005",**

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

\]

}

}

\]

}

This way, all entities providing temperature values within SocIoTal have been discovered, and their corresponding IDs have been obtained to be further requested about their stored values. Using an empty set of attributes (by not providing an “attributes” element) in the request, the discovering process will return all context information available (the complete set of entities with their associated attributes) related to SocIoTal. This way, with the ID’s obtained, the SocIoTal CM can be queried in order to obtain the context information discovered. It can be done one entity by one or they can be grouped, now the IDs are known, using patterns to request in one query, as in the queryContext shown below.

| SocIoTal SERVER                                                                            | platform.sociotal.eu:3500                        |
|--------------------------------------------------------------------------------------------|--------------------------------------------------|
| POST                                                                                       | /SocIoTal\_CM\_REST\_V3/NGSI10\_API/queryContext |
| HEADERS                                                                                    |                                                  |
|                                                                                            | Content-type                                     |
|                                                                                            | Accept                                           |
| If required                                                                                | Capability-token                                 |
| If required                                                                                | Community-Token                                  |
| *If required*                                                                              | *X-Auth-Token*                                   |
| queryContextRequest                                                                        |
| {                                                                                          
                                                                                             
 "entities": \[{                                                                             
                                                                                             
 "type": "",                                                                                 
                                                                                             
 "isPattern": "true",                                                                        
                                                                                             
 "id": "SocIoTal:SAN:WeatherStation:Dev\_00.\*" //Pattern that fits all discovered entities  
                                                                                             
 }\],                                                                                        
                                                                                             
 "attributes":\["AmbientTemperature"\] //Context Info we’re querying for                     
                                                                                             
 }                                                                                           |

So, the result will be:

queryContextResponse

{

"errorCode": {

"details": "Count: 3",

"reasonPhrase": "OK",

"code": "200"

},

"contextResponses": \[

{

"statusCode": {

"reasonPhrase": "OK",

"code": "200"

},

"contextElement": {

"id": "SocIoTal:SAN:WeatherStation:Dev\_001",

"attributes": \[

{

"name": "AmbientTemperature",

"value": "22.47",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature",

"metadatas": \[

{

"name": "DateTimeStamp",

"value": "20150515T113739Z",

"type": "http://sensorml.com/ont/swe/property/DateTimeStamp"

},

{

"name": "Unit",

"value": "celsius",

"type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"

},

{

"name": "accuracy",

"value": "0,25",

"type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"

},

{

"name": "DataDescription",

"value": "float",

"type": "http://sensorml.com/ont/swe/property/DataDescription"

}

\]

}

\],

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

},

{

"statusCode": {

"reasonPhrase": "OK",

"code": "200"

},

"contextElement": {

"id": "SocIoTal:SAN:WeatherStation:Dev\_002",

"attributes": \[

{

"name": "AmbientTemperature",

"value": "21.35",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature",

"metadatas": \[

{

"name": "DateTimeStamp",

"value": "20150511T090815Z",

"type": "http://sensorml.com/ont/swe/property/DateTimeStamp"

},

{

"name": "Unit",

"value": "celsius",

"type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"

},

{

"name": "accuracy",

"value": "0,25",

"type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"

},

{

"name": "DataDescription",

"value": "float",

"type": "http://sensorml.com/ont/swe/property/DataDescription"

}

\]

}

\],

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

},

{

"statusCode": {

"reasonPhrase": "OK",

"code": "200"

},

"contextElement": {

"id": "SocIoTal:SAN:WeatherStation:Dev\_005",

"attributes": \[

{

"name": "AmbientTemperature",

"value": "20.5",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature",

"metadatas": \[

{

"name": "DateTimeStamp",

"value": "20150126T095005Z",

"type": "http://sensorml.com/ont/swe/property/DateTimeStamp"

},

{

"name": "Unit",

"value": "celsius",

"type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"

},

{

"name": "accuracy",

"value": "0,25",

"type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"

},

{

"name": "DataDescription",

"value": "float",

"type": "http://sensorml.com/ont/swe/property/DataDescription"

}

\]

}

\],

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

}

\]

}

Above, all current available Ambient Temperature context information provided by SocIoTal entities is described, including last captured values and the metadata referred to those values.

### subscribeContextAvailability

Allows the asynchronous discovery of the potential set of context entities belonging to an identified type and/or providing desired attributes. This is, by using this method, we will be able to receive (by POST method) an asynchronous notification (to the “reference” registered callback function) when a new entity is added to SocIoTal group (or an existing one changes), when a new device appears or when a new “AmbientTemperature” attribute is added (by a new entity or an existing one). Also, different options can be combined, in order to subscribe e.g. to devices registered within SocIoTal and providing “AmbientTemperature” information. Next method call subscribes [**http://capela.unican.es:1650/Callback\_Apps/Listeners/LogWriterFile**](http://capela.unican.es:1650/Callback_Apps/Listeners/LogWriterFile) application (just a simple listener) to all “AmbientTemeperature” and/or “HumidityValue” context belonging to SocIoTal.

| SocIoTal SERVER                                                                     | platform.sociotal.eu:3500                                       |
|-------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| POST                                                                                | /SocIoTal\_CM\_REST\_V3/NGSI9\_API/subscribeContextAvailability |
| HEADERS                                                                             |                                                                 |
|                                                                                     | Content-type                                                    |
|                                                                                     | Accept                                                          |
| If required                                                                         | Capability-token                                                |
| If required                                                                         | Community-Token                                                 |
| *If required*                                                                       | *X-Auth-Token*                                                  |
| subscribeContextAvailabilityRequest                                                 |
| {                                                                                   
                                                                                      
 "entities": \[{                                                                      
                                                                                      
 "type": "urn:x-org:sociotal:resource:device",                                        
                                                                                      
 "isPattern": "true",                                                                 
                                                                                      
 "id": "SocIoTal.\*"                                                                  
                                                                                      
 }\],                                                                                 
                                                                                      
 "attributes": \["AmbientTemperature", "HumidityValue"\],                             
                                                                                      
 "reference": "http://capela.unican.es:1650/Callback\_Apps/Listeners/LogWriterFile",  
                                                                                      
 "duration": "P1M"                                                                    
                                                                                      
 }                                                                                    |

This request will return a subscriptionId which will identify the context subscription.

subscribeContextAvailabilityResponse

{

"duration": "P1M",

"subscriptionId": "**5559c7d1d9e4be3111e2a18b**"

}

This subscriptionId will be further used to update (or modify) the subscription (e.g. renew the duration parameter, extending the lifetime of the subscription) or to delete it.

As the response (first response), “reference” (application subscribed) element will be posted (http POST) with a payload including all entities currently subscribed, belonging to SocIoTal deployment that provides either AmbientTemperature, HumidityValue or both attributes.

notifyContextAvailability

{

"contextRegistrationResponses": \[{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

},

{

"isDomain": "false",

"name": "HumidityValue",

"type": "http://sensorml.com/ont/swe/property/HumidityValue"

}\],

"entities": \[{

"id": "SocIoTal:SAN:WeatherStation:Dev\_001",

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}\]

}

},

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

}\],

"entities": \[{

"id": "SocIoTal:SAN:WeatherStation:Dev\_002",

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}\]

}

},

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

},

{

"isDomain": "false",

"name": "HumidityValue",

"type": "http://sensorml.com/ont/swe/property/HumidityValue"

}\],

"entities": \[{

"id": "SocIoTal:CRS4:WeatherStation:Dev\_005",

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}\]

}

},

{

"contextRegistration": {

"providingApplication": "http://platform.sociotal.eu:3500/SocIoTal\_CM\_REST\_V3/NGSI10\_API",

"attributes": \[{

"isDomain": "false",

"name": "AmbientTemperature",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature"

},

{

"isDomain": "false",

"name": "HumidityValue",

"type": "http://sensorml.com/ont/swe/property/HumidityValue"

}\],

"entities": \[{

"id": "SocIoTal:SAN:WeatherStation:Dev\_004",

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}\]

}

}\],

"subscriptionId": "**5559c7d1d9e4be3111e2a18b**"

}

Next time a new entity (or existing one) registers a new AmbientTemperature and/or HumidityValue attribute, a similar message, including only the new context information will be sent to the subscribed app/resource.

### subscribeContext

So far, the discovery of entities (either resources, devices, etc.) has been tested based on the context they can provide, including also the synchronous request of the discovered context information. This last test is related to discover changes in the interesting context information. Using SocIoTal CM subscribeContext method, the requestor will receive an asynchronous notification every time the selected context changes or a synchronous one, including the current values, every time a defined time slot expires:

-   ONTIMEINTERVAL: will send a notification when the time interval specified in the “notifyConditions” element is reached.

-   ONCHANGE: provides a notification every time a change in the specified context attributes happens.

| SocIoTal SERVER                                                                     | platform.sociotal.eu:3500                            |
|-------------------------------------------------------------------------------------|------------------------------------------------------|
| POST                                                                                | /SocIoTal\_CM\_REST\_V3/NGSI10\_API/subscribeContext |
| HEADERS                                                                             |                                                      |
|                                                                                     | Content-type                                         |
|                                                                                     | Accept                                               |
| If required                                                                         | Capability-token                                     |
| If required                                                                         | Community-Token                                      |
| *If required*                                                                       | *X-Auth-Token*                                       |
| subscribeContextRequest: \[ONCHANGE\]                                               |
| {                                                                                   
                                                                                      
 "entities": \[{                                                                      
                                                                                      
 "type": " urn:x-org:sociotal:resource:device ",                                      
                                                                                      
 "isPattern": "false",                                                                
                                                                                      
 "id": "SocIoTal:SAN:WeatherStation:Dev\_001"                                         
                                                                                      
 }\],                                                                                 
                                                                                      
 "reference": "http://capela.unican.es:1650/Callback\_Apps/Listeners/LogWriterFile",  
                                                                                      
 "duration": "P1M",                                                                   
                                                                                      
 "notifyConditions": \[{                                                              
                                                                                      
 "type": "ONCHANGE",                                                                  
                                                                                      
 "condValues": \["AmbientTemperature"\]                                               
                                                                                      
 }\],                                                                                 
                                                                                      
 "throttling": "PT5S"                                                                 
                                                                                      
 }                                                                                    |
| subscribeContextRequest:: \[ONTIMEINTERVAL\]                                        |
| {                                                                                   
                                                                                      
 "entities": \[{                                                                      
                                                                                      
 "type": "urn:x-org:sociotal:resource:device",                                        
                                                                                      
 "isPattern": "false",                                                                
                                                                                      
 "id": "SocIoTal:SAN:WeatherStation:Dev\_001"                                         
                                                                                      
 }\],                                                                                 
                                                                                      
 "attributes": \["HumidityValue"\],                                                   
                                                                                      
 "reference": "http://capela.unican.es:1650/Callback\_Apps/Listeners/LogWriterFile",  
                                                                                      
 "duration": "P1M",                                                                   
                                                                                      
 "notifyConditions": \[{                                                              
                                                                                      
 "type": "ONTIMEINTERVAL",                                                            
                                                                                      
 "condValues": \["PT10S"\]                                                            
                                                                                      
 }\]                                                                                  
                                                                                      
 }                                                                                    |

Within these requests:

-   **“entities”** and **“attributes”** define which context elements will be included in the notification message. They work the same way as in the queryContext request, so lists of attributes or patterns in the entity id can be included to specify a set of entities and/or attributes the requestor would like to be subscribed.

-   **“reference”** element specifies the callback function where the SocIoTal CM is going to send the notifications (when happen). It will POST a JSON object including the context information to the URL pointed here.

-   **“duration”** gives the subscription duration, specified using the ISO 8601 \[40\] standard format, for example “P1M” means 1 moth.

-   **“condValues”** specifies:

    -   ONCHANGE subscription: the attribute or set of attributes that, when changing, triggers a notification. It is important to mention that the set of attributes pointed here will be those that just trigger the action, but not necessarily those to be included in the notification to be sent. The ones to be included in the notified context information will be specified in an “attributes” element. If none are specified (the “attributes” element is not included in the payload, as in the one requested here) all available attribute values (all context information) compliant with the rest of pointed conditions will be retrieved.

    -   ONTIMEINTERVAL subscription: the time between received notifications.

-   **“throttling”** specifies a minimum inter-notification arrival time. So, setting throttling to 5 seconds as in the example above makes that a notification will not be sent if a previous notification was sent less than 5 seconds ago, no matter how many actual changes take place in that period.

| subscribreContextResponse                        |
|--------------------------------------------------|
| {                                                
                                                   
 "subscribeResponse": {                            
                                                   
 "duration": "P1M",                                
                                                   
 "throttling": "PT5S",                             
                                                   
 "subscriptionId": "**5559e2f9d9e4be3111e2a18e**"  
                                                   
 }                                                 
                                                   
 }                                                 |

The response given by the SocIoTal context manager will include the subscription ID, used lately to modify and/or delete it.

The request (ONCHANGE) above has subscribed [**http://capela.unican.es:1650/Callback\_Apps/Listeners/LogWriterFile**](http://capela.unican.es:1650/Callback_Apps/Listeners/LogWriterFile) to WS:Dev\_001 context information. Every time a change in the AmbientTemperature of this entity is registered in the SocIoTal CM, a notification including all context information available from WS:Dev\_001 will be posted to this resource, as the one shown below.

notifyContextAvailability

{

"contextResponses": \[{

"statusCode": {

"reasonPhrase": "OK",

"code": "200"

},

"contextElement": {

"id": "SocIoTal:SAN:WeatherStation:Dev\_001",

"attributes": \[{

"name": "AmbientTemperature",

"value": "24.70",

"type": "http://sensorml.com/ont/swe/property/AmbientTemperature",

"metadatas": \[{

"name": "DateTimeStamp",

"value": "20150401T093702Z",

"type": "http://sensorml.com/ont/swe/property/DateTimeStamp"

},

{

"name": "Unit",

"value": "celsius",

"type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"

},

{

"name": "accuracy",

"value": "0,25",

"type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"

},

{

"name": "DataDescription",

"value": "float",

"type": "http://sensorml.com/ont/swe/property/DataDescription"

}\]

},

{

"name": "HumidityValue",

"value": "56",

"type": "http://sensorml.com/ont/swe/property/HumidityValue",

"metadatas": \[{

"name": "DateTimeStamp",

"value": "20150401T093702Z",

"type": "http://sensorml.com/ont/swe/property/DateTimeStamp"

},

{

"name": "Unit",

"value": "percentage",

"type": "http://purl.oclc.org/NET/ssnx/qu/qu\#Unit"

},

{

"name": "accuracy",

"value": "1",

"type": "http://sensorml.com/ont/swe/property/QuantitativeAttributeAccuracy"

},

{

"name": "DataDescription",

"value": "integer",

"type": "http://sensorml.com/ont/swe/property/DataDescription"

}\]

},

{

"name": "Location",

"value": "43.472057, -3.800156",

"type": "http://sensorml.com/ont/swe/property/Location",

"metadatas": \[{

"name": "WorldGeographicReferenceSystem",

"value": "WGS84",

"type": "http://sensorml.com/ont/swe/property/WorldGeographicReferenceSystem"

}\]

}\],

"type": "urn:x-org:sociotal:resource:device",

"isPattern": "false"

}

}\],

"originator": "localhost",

"subscriptionId": "**5559e2f9d9e4be3111e2a18e**"

}
