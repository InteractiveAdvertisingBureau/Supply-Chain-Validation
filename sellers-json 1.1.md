# Sellers.json

### Abstract 

As part of a broader effort to eliminate the ability to profit from counterfeit inventory in the open digital advertising ecosystem, Sellers.json provides a mechanism to enable buyers to discover who the entities are that are either direct sellers of or intermediaries in the selling of digital advertising. 

### About IAB Tech Lab 

The IAB Technology Laboratory is a nonprofit research and development consortium charged with producing and helping companies implement global industry technical standards and solutions. The goal of the Tech Lab is to reduce friction associated with the digital advertising and marketing supply chain while contributing to the safe growth of an industry.  
 
The IAB Tech Lab spearheads the development of technical standards, creates and maintains a code library to assist in rapid, cost-effective implementation of IAB standards, and establishes a test platform for companies to evaluate the compatibility of their technology solutions with IAB standards, which for 18 years have been the foundation for interoperability and profitable growth in the digital advertising supply chain. Further details about the IAB Technology Lab can be found at [https://iabtechlab.com](https://iabtechlab.com). 
 
### License 

Sellers.json by OpenRTB Working Group is licensed under a Creative Commons Attribution 3.0 License.   To view a copy of this license, visit creativecommons.org/licenses/by/3.0/ or write to Creative Commons, 171 Second Street, Suite 300, San Francisco, CA 94105, USA. 
 
### Disclaimer 
THE STANDARDS, THE SPECIFICATIONS, THE MEASUREMENT GUIDELINES, AND ANY OTHER MATERIALS OR SERVICES PROVIDED TO OR USED BY YOU HEREUNDER (THE “PRODUCTS AND SERVICES”) ARE PROVIDED “AS IS” AND “AS AVAILABLE,” AND IAB TECHNOLOGY LABORATORY, INC. (“TECH LAB”) MAKES NO WARRANTY WITH RESPECT TO THE SAME AND HEREBY DISCLAIMS ANY AND ALL EXPRESS, IMPLIED, OR STATUTORY WARRANTIES, INCLUDING, WITHOUT LIMITATION, ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AVAILABILITY, ERROR-FREE OR UNINTERRUPTED OPERATION, AND ANY WARRANTIES ARISING FROM A COURSE OF DEALING, COURSE OF PERFORMANCE, OR USAGE OF TRADE. TO THE EXTENT THAT TECH LAB MAY NOT AS A MATTER OF APPLICABLE LAW DISCLAIM ANY IMPLIED WARRANTY, THE SCOPE AND DURATION OF SUCH WARRANTY WILL BE THE MINIMUM PERMITTED UNDER SUCH LAW. THE PRODUCTS AND SERVICES DO NOT CONSTITUTE BUSINESS OR LEGAL ADVICE. TECH LAB DOES NOT WARRANT THAT THE PRODUCTS AND SERVICES PROVIDED TO OR USED BY YOU HEREUNDER SHALL CAUSE YOU AND/OR YOUR PRODUCTS OR SERVICES TO BE IN COMPLIANCE WITH ANY APPLICABLE LAWS, REGULATIONS, OR SELF-REGULATORY FRAMEWORKS, AND YOU ARE SOLELY RESPONSIBLE FOR COMPLIANCE WITH THE SAME, INCLUDING, BUT NOT LIMITED TO, DATA PROTECTION LAWS, SUCH AS THE PERSONAL INFORMATION PROTECTION AND ELECTRONIC DOCUMENTS ACT (CANADA), THE DATA PROTECTION DIRECTIVE (EU), THE E-PRIVACY DIRECTIVE (EU), THE GENERAL DATA PROTECTION REGULATION (EU), AND THE E-PRIVACY REGULATION (EU) AS AND WHEN THEY BECOME EFFECTIVE. 
 
## Table of Contents 
 
- [Abstract	](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#abstract) 
- [About IAB Tech Lab](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#about-iab-tech-lab)	
    - [Introduction](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#introduction)	
    - [Change Log	](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#change-log)
- [SPECIFICATION](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#specification)	
    - [ACCESS METHOD](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#access-method)	
    - [FILE FORMAT](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#file-format)	
    - [EXPIRATION](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#expiration)
    - [Proposed implementation](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#implementation)	
    - [Object specifications](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#object-specifications)	
        - [Object: Parent](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#object-parent)	
        - [Sellers.json object: Identifier](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#sellersjson-object-identifier)	
        - [Sellers.json object: Seller	](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#sellersjson-object-seller)
    - [Enumerated List Specification	](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#enumerated-list-specification)
        - [Identifier Names](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#identifier-names)	
    - [Sample File Contents](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#sample-file-contents) 

## Introduction 

Ads.txt has been extremely successful in allowing publisher content distributors and app publishers to define who is authorized to sell a given set of impressions (publisher’s advertising inventory) via a bid request. Ads.txt does not, however, make any attempt at revealing the identities of the publisher account IDs within their advertising platform(s) (Publisher's SSPSupply-Side Platform).  
 
A published and accessible Sellers.json file allows the identity of the final seller of a bid request to be discovered (assuming that they are ads.txt authorized).  It also allows the identities of all nodes (entities that participated in the bid request) in the [SupplyChain](https://iabtechlab.com/sellers-json/) object to be discovered. Currently, it is possible for the final seller to be identified via the Publisher.name and Publisher.domain attributes, but in practice, these properties are inconsistently populated by various selling systems.  
 
Sellers.json enables smaller bid request object sizes by allowing this information to be looked up and cached “offline” rather than supplied with every bid request. Sellers.json also allows the identification of any and all intermediaries that participated in the selling of a bid request. 
 
### Audience 

Advertising systems, whether they are sellers or buyers of programmatic advertising both benefit from a transparent marketplace where all parties to the transaction are well understood. This document serves to define the information that sellers will provide to both downstream sellers and buyers of programmatic inventory. 

### Change Log 

|Version |	Date |	Changes |
|---|---|---|
|1.0 |	July 2019 |	First Version |
 
### SPECIFICATION 
 
This document specifies a mechanism for advertising systems to publicly declare the sellers that they represent and those sellers’ identifiers within their systems. It also describes the format for encoding the instructions to be consumed by advertising systems and their customers.  

### ACCESS METHOD  

Advertising systems should post the sellers.json file on their root domain and any subdomains as needed.  For the purposes of this document the “root domain” is defined as the “public suffix” plus one string in the name. Crawlers should incorporate Public Suffix list [16] to derive the root domain. 
 
The declarations must be accessible via HTTP and/or HTTPS from the website where the instructions are to be applied under a standard relative path on the server host: "/sellers.json" and HTTP request header containing "Content-Type: application/json". Additionally, you could use "Content-Type: application/json; charset=utf-8" to explicitly signal UTF8 support.   
 
Also, HTTPS connections over HTTP are preferred when crawling sellers.json files. In any case, where data is available at both an HTTPS and an HTTP connection for the same URL, the data from HTTPS should be preferred. 
 
For convenience we will refer to this resource as the "/sellers.json" file. Despite the use of the word "file," the resource need not originate from a file system.  
 
If the server response indicates Success (HTTP 2xx Status Code,) the advertising system must read the content, parse it, and utilize the declarations. 
 
If the server response indicates an HTTP/HTTPS redirect (301, 302, 307 status codes), the advertising system should follow the redirect and consume the data as authoritative for the source of the redirect, if and only if the redirect is within the scope of the original root domain defined as the public suffix plus one string in the name as defined above.  Multiple redirects are valid as long as each redirect location remains within the original root domain.  For example an HTTP to HTTPS redirect within the same root domain is valid.   
 
Only a single HTTP redirect to a destination outside the original root domain is allowed to facilitate one-hop delegation of authority to a third party's web server domain. If the third party location returns a redirect, then the advertising system should treat the response as an error.  A future version may address other delegation of authority to a third-party web server, but for now redirects to a third-party web server and any other redirect should be interpreted as an error and ignored. 
 
If the server response indicates the resource does not exist (HTTP Status Code 404) or for any other HTTP error, the last successfully retrieved data set should be utilized. 
 
### FILE FORMAT 

All data in the file is serialized using JSON (JavaScript Object Notation). The parent JSON object and all child objects are written to the sellers.json file.   

### EXPIRATION 

Consuming systems of /sellers.json should cache the files, but if they do they must periodically verify the cached copy is fresh before using its contents. 
 
Standard HTTP cache-control mechanisms can be used by both origin server and robots to influence the caching of the /sellers.json file.  Specifically consumers and replicators should take note of HTTP Expires header set by the origin server. 
 
If no cache-control directives are present, consuming systems should default to an expiry of 7 days. 

## Implementation 

Every advertising system listed in an ads.txt file and any advertising system that is referenced from a SupplyChain object node should publish a Sellers.json file at the following location: 
 
`http://{advertising_system_domain}/sellers.json` 
 
## Object specifications 

### Object: Parent

The Parent object is the top-level object of a Sellers.json file. It is a container for all properties in a sellers.json file. 
 
 
|Attribute |	Type |	Description|
|---|---|---|
|sellers |	object array; required |	The list of all Seller objects that are represented by the advertising system. All sellers must be included even if they are confidential. |
|identifiers |	object array; optional |	Array of Identifier objects associated with this advertising system. Examples could be TAG-Ids, Dun & Bradstreet business identifiers, or any custom identifier that a consuming advertising system might need.| 
|contact_email |	string; optional |	An email address to use to contact the Advertising System for questions or inquiries about this file. |
|contact_address |	string; optional |	The business address of the advertising system. |
|version |	string; required |	The version of this spec, currently the only valid value is 1.0. |
|ext |	object; optional |	Placeholder for advertising-system specific extensions to this object.| 
 
## Sellers.json object: Identifier 

An identifier is an arbitrary name/value pair that is used to communicate common values such as business identifiers, certification identifiers, or any other identifier that a consuming system might need to better interoperate with the seller. 
 
Standard Identifiers are listed in the section [Identifier Names](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/sellers-json.md#identifier-names) below. 
 
|Attribute |	Type |	Description |
|---|---|---|
|name |	string; required |	The description of the identifier |
|value |	string; required |	The value of the identifier |
 
### Sellers.json object: Seller 

The identification of the selling legal entity that is paid for inventory sold on behalf of seller_id, or — for passthrough sellers (see `is_passthrough`) — the legal entity on whose behalf bid requests are transacted under the seller_id. 
 
It is invalid for a seller_id to represent multiple entities. Every seller_id must map to only a single entity that is paid for inventory transacted with that seller_id. It is valid for a selling entity to have multiple seller_ids within an advertising system. Where the same entity transacts with an advertising system under both a paid relationship and a passthrough relationship concurrently (for example, an advertising system that acts as a payment intermediary for some demand partners and as a technology provider for demand partners that pay the entity directly), separate seller_ids should be used for each relationship, so that each seller_id unambiguously describes one relationship.  

|Attribute |	Type |	Description |
|---|---|---|
|seller_id |	string; required |	This is the same ID that appears in an ads.txt file and in the SupplyChain.nodes array sid property. In most cases will also appear in the Publisher.Id property of an OpenRTB request. |
|is_confidential |	integer; optional, default 0 |	Indicates whether the identity of the seller is confidential, where 0 = is not confidential and 1 = is confidential. |
|seller_type |	string; required |	An enumeration of the type of account, either PUBLISHER, INTERMEDIARY, or BOTH. <br><br>A value of "PUBLISHER" indicates that the inventory sold through this account is on a site, app, or other medium owned by the named entity and the advertising system pays them directly. <br><br>A value of “INTERMEDIARY" indicates that the inventory sold through this account is not owned by the named entity or the advertising system does not pay them directly. All intermediaries take custody of the bid request. Intermediaries may or may not handle payment. <br><br>'BOTH' indicates that both types of inventory are transacted by this seller. Note that this field should be treated as case insensitive when interpreting the data. |
|is_passthrough |	integer; optional, default 0 |	A passthrough seller is a facilitator of inventory from the upstream supplier to the consumer of the inventory. The upstream supplier and consumer must establish a business relationship with each other such that the upstream supplier has control of their account within the consumer’s platform. <br><br> **A value of 1** indicates the following: <br>- This seller has an account control relationship with the downstream/consuming advertising system. <br>-	If this seller is the last link in a SupplyChain, the buying system has to have established an account control relationship with this seller to transact the seller’s inventory. <br>-	If this is not the last link in a SupplyChain than this seller should exist between two entities that have an account control relationship. <br><br> This should be set to 1 for the seller to be used as a non-payment node (hp=0) in Object: Supply Chain.<br><br> **A value of 0** indicates: <br>- The downstream/consuming advertising system has no account control relationship with this seller.  |
|name |	string; required when is_confidential=0 |	The name of the company (the legal entity) that is paid for inventory transacted under the given seller_id, or — for passthrough sellers — on whose behalf requests are transacted under the given seller_id. Can be omitted only when is_confidential is set to 1.| 
|domain |	string; required if seller has a web presence and is_confidential=0 |	The business domain name of the company (the legal entity) that is paid for inventory transacted under the given seller_id, or — for passthrough sellers — on whose behalf requests are transacted under the given seller_id. When the seller_type property is set to INTERMEDIARY or BOTH, this should be the root domain name of the seller’s Sellers.json file. Can be omitted when is_confidential is set to 1 or when the seller doesn’t have a web presence.  |
|comment |	string; optional |	Any helpful description for this inventory. It is useful for sellers that have multiple seller ids to describe what this seller_id represents. |
|ext |	object; optional |	Placeholder for advertising-system specific extensions to this object. |
 
Note that domain in sellers.json and SupplyChain are to be populated with only the **root domain** and not a full URL, much like ‘domain’ in OpenRTB’s Site object and the domain definition used for ads.txt. For the purposes of this document the “root domain” is defined as the “public suffix” plus one string in the name. Implementers should incorporate Public Suffix list [16] to derive the root domain. 

## Enumerated List Specification 

### Identifier Names 

The following list defines standard identifiers that should be used in the identifier list. 
 
|Name |	Description |
|---|---|
|TAG-ID |	Trustworthy Accountability Group ID | 
|DUNS |	Dun & Bradstreet DUNS Number |
 
## Sample File Contents 

```json
{ 
  "contact_email": "adops@advertisingsystem.com", 
  "contact_address": "Advertising System Inc., 101 Main Street, New York, NY 10101", 
  "version": "1.0", 
  "identifiers": [ 
    { 
      "name": "TAG-ID", 
      "value": "28cb65e5bbc0bd5f" 
    } 
  ], 
  "sellers": [ 
    { 
      "seller_id": "1942009976", 
      "name": "Publisher1", 
      "domain": "publisher1.com", 
      "seller_type": "PUBLISHER” 
    }, 
    { 
      "seller_id": "1397382429", 
      "name": "Exchange1", 
      "domain": "exchange1.com", 
      "seller_type": "INTERMEDIARY" 
    }, 
    { 
      "seller_id": "20000000", 
      "name": "Seller And Intermediary, Inc", 
      "domain": "sellerandintermediary.com", 
      "seller_type": "PUBLISHER", 
      "comment": "NorthAmerica O&O inventory" 
    }, 
    { 
      "seller_id": "20000001", 
      "name": "Seller And Intermediary, Inc", 
      "domain": "sellerandintermediary.com", 
      "seller_type": "PUBLISHER", 
      "comment": "APAC O&O inventory" 
    }, 
    { 
      "seller_id": "20000002", 
      "name": "Seller And Intermediary, Inc", 
      "domain": "sellerandintermediary.com", 
      "seller_type": "INTERMEDIARY", 
      "comment": "Non O&O inventory" 
    }, 
    { 
      "seller_id": "101010101", 
      "name": "Hybrid Seller", 
      "domain": "hybridseller.com", 
      "seller_type": "BOTH", 
      "comment": "Sells both O&O and other sellers' inventory" 
    }, 
    { 
      "seller_id": "00000001", 
      "seller_type": "INTERMEDIARY", 
      "is_confidential": 1 
    }, 
    { 
      "seller_id": "EB_0001", 
      "name": "Passthrough Publisher", 
      "domain": "passthroughpublisher.com", 
      "seller_type": "PUBLISHER", 
      "is_passthrough": 1,       "comment": "direct buyer/seller of this inventory must establish an account relationship with Passthrough Publisher"     } 
  ] 
} 
```
