# Supply Chain Validation: Implementation Guidance

This document consolidates three source references into a single implementation guide for publishers, SSPs, exchanges, DSPs, and ad servers implementing sellers.json, the SupplyChain object, and ads.txt inventory sharing.

**Sources:**
- [OpenRTB SupplyChain Object Specification](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/develop/2.6.md#3225---object-supplychain-) — IAB Tech Lab
- [OpenRTB Supply Chain Object Implementation (DEPRECATED)](https://github.com/InteractiveAdvertisingBureau/openrtb/blob/main/supplychainobject.md)
- [Ads.txt & App-ads.txt: Guidance for Inventory Sharing (Explainer Guide)](https://github.com/InteractiveAdvertisingBureau/Supply-Chain-Validation/blob/main/Explainer%20Guide.md) — IAB Tech Lab
- FAQ for sellers.json and SupplyChain Object (July 2019, updated September 2020) — IAB Tech Lab

---

## Table of Contents
1. [Overview](#1-overview)
2. [OpenRTB SupplyChain Object Specification](#openrtb-supplychain-object-specification)
   - [Implementation Rules](#openrtb-supply-chain-object-implementation-rules)
   - [Placement in the Bid Request](#placement-in-the-bid-request)
   - [OpenRTB Examples — Version 1.0](#openrtb-examples---version-10)
   - [OpenRTB Examples — Version 1.1](#openrtb-examples---version-11)
     - [WEB-1: Device > Server-Side Header Bidding Wrapper > SSP > DSP](#web-1-device--server-side-header-bidding-wrapper--ssp--dsp)
     - [WEB-2: Device > Open Bidding Provider > SSP > DSP](#web-2-device--open-bidding-provider--ssp--dsp)
     - [WEB-3: Device > Client-Side Wrapper > Server-Side Header Bidding Wrapper > SSP > DSP](#web-3-device--client-side-wrapper--server-side-header-bidding-wrapper--ssp--dsp)
     - [WEB-4: Device > Prebid JS > SSP > DSP](#web-4-device--prebid-js--ssp--dsp)
     - [WEB-5: Device > Publisher Ad Management JS > Publisher Ad Management Server > SSP > DSP](#web-5-device--publisher-ad-management-js--publisher-ad-management-server--ssp--dsp)
     - [WEB-6: Device > Publisher Ad Management JS > Yield Optimization Intermediary > SSP > DSP](#web-6-device--publisher-ad-management-js--yield-optimization-intermediary--ssp--dsp)
     - [MOB-1: Client Device > Mobile SDK > DSP](#mob-1-client-device--mobile-sdk--dsp)
     - [MOB-2: Client Device > Mobile SDK > SSP > DSP (Classic Resell)](#mob-2-client-device--mobile-sdk--ssp--dsp-classic-resell)
     - [MOB-3: Client Device > Mobile SDK > SSP > DSP (SDK Renting)](#mob-3-client-device--mobile-sdk--ssp--dsp-sdk-renting)
     - [CTV-1: Device > SSAI Platform > CTV Ad Server > SSP > DSP](#ctv-1-device--ssai-platform--ctv-ad-server--ssp--dsp-canonical-single-path)
     - [CTV-2: Device > Pure SSAI Vendor > CTV Ad Server > SSP > DSP](#ctv-2-device--pure-ssai-vendor--ctv-ad-server--ssp--dsp)
     - [CTV-3: Device > SSAI + Ad Server (Same Operator) > SSP > DSP](#ctv-3-device--ssai--ad-server-same-operator--ssp--dsp)
     - [CTV-4: Device > Publisher-Owned SSAI > Primary Ad Server > Content Owner Ad Server > SSP > DSP](#ctv-4-device--publisher-owned-ssai--primary-ad-server--content-owner-ad-server--ssp--dsp)
     - [CTV-5a: Device > SSAI > App Owner Ad Server > SSP > DSP (App Owner Path)](#ctv-5a-device--ssai--app-owner-ad-server--ssp--dsp-app-owner-path--inventory-share)
     - [CTV-5b: Device > SSAI > Inventory Share Partner Ad Server > SSP > DSP](#ctv-5b-device--ssai--inventory-share-partner-ad-server--ssp--dsp-inventory-share-partner-path)
     - [CTV-6: Device > SSAI Platform > Content Owner Ad Server > Primary CTV Ad Server > SSP > DSP](#ctv-6-device--ssai-platform--content-owner-ad-server--primary-ctv-ad-server--ssp--dsp)
     - [CTV-7: Device > SSAI Platform > Content Owner Ad Server > Primary CTV Ad Server > SSP > DSP (Sequential)](#ctv-7-device--ssai-platform--content-owner-ad-server--primary-ctv-ad-server--ssp--dsp-sequential-ad-server-connections)
     - [CTV-8: Device > SSAI Platform > Primary CTV Ad Server > SSP-1 > SSP-2 > DSP (SSP Chaining)](#ctv-8-device--ssai-platform--primary-ctv-ad-server--ssp-1--ssp-2--dsp-ssp-chaining)
   - [Non-OpenRTB Tag Serialization](#26-non-openrtb-tag-serialization)
3. [Inventory Sharing: Ads.txt & App-ads.txt Explainer](#3-inventory-sharing-adstxt--app-adstxt-explainer)
   - [Background](#31-background)
   - [Scope](#32-scope)
   - [Updates to the Standard](#33-updates-to-the-standard)
   - [Example Use Cases](#34-example-use-cases)
   - [Use Case Implementation Logic](#35-use-case-implementation-logic)
   - [Implementation Guidelines for CTV/OTT](#36-implementation-guidelines-for-ctvott)
4. [FAQ: sellers.json and SupplyChain Object](#4-faq-sellersjson-and-supplychain-object)
   - [What Are sellers.json and SupplyChain?](#41-what-are-sellersjson-and-supplychain)
   - [VAST and Tag-Based Requests](#42-vast-and-tag-based-requests)
   - [SSAI Vendors](#43-ssai-vendors)
   - [Correctness and Fraud](#44-correctness-and-fraud)
   - [Seller ID](#45-seller-id)
   - [Validation Methods](#46-validation-methods)
   - [Publisher Responsibilities](#47-publisher-responsibilities)
   - [Setting seller_type](#48-setting-seller_type)
   - [Setting is_passthrough](#49-setting-is_passthrough)
   - [Header Bidding](#410-header-bidding)
   - [Worked Examples](#411-worked-examples)

---

## 1. Overview

sellers.json and the SupplyChain object are complementary mechanisms for disclosing the full chain of entities involved in the sale of a programmatic ad impression — specifically those who took custody of a bid request.

**sellers.json** is a file published by each advertising system at `{domain}/sellers.json`. It declares every seller or intermediary the advertising system represents, identified by a `seller_id`. Buyers can look up any node in a SupplyChain object against the corresponding advertising system's sellers.json to verify the identity of that entity.

**The SupplyChain object** travels with the bid request. It lists every node in the chain from the originating seller to the entity sending the current bid request. Each node identifies the advertising system (`asi`) and the seller's account on that system (`sid`), and indicates whether that node is in the direct payment chain (`hp`).

**ads.txt / app-ads.txt** provide the publisher-side authorization record: which advertising systems are permitted to sell a publisher's inventory, and under what relationship (DIRECT or RESELLER). The `inventorypartnerdomain` directive extends this for CTV/OTT inventory sharing scenarios where multiple entities have ownership rights over ad space within the same app or site.

Together, these three mechanisms allow a DSP to answer: *Who authorized the sale of this impression, who is selling it, and is every entity in the chain one I want to transact with?*

---

## OpenRTB SupplyChain Object Specification

*Source: [IAB Tech Lab OpenRTB SupplyChain Object](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/develop/2.6.md#3225---object-supplychain-)*

As part of a broader effort to eliminate the ability to profit from invalid traffic, ad fraud, and counterfeit inventory in the open digital advertising ecosystem, the SupplyChain object enables buyers to see all parties who have taken custody of a bid request.

Ads.txt has been extremely successful in allowing publishers and app makers to define who is authorized to sell a given set of impressions via the programmatic marketplace. Ads.txt does not however make any attempt at revealing or authorizing all parties that are part of the transacting of those impressions. This information can be important to buyers for a number of reasons including transparency of the supply chain, ensuring that all intermediaries are entities with which the buyer wants to transact and that inventory is purchased as directly as possible.

The [SupplyChain object](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/develop/2.6.md#3225---object-supplychain-) is composed primarily of a set of nodes where each node represents a specific entity that participates in the transacting of inventory. The entire chain of nodes using [Object: SupplyChainNode](https://github.com/InteractiveAdvertisingBureau/openrtb2.x/blob/develop/2.6.md#3226---object-supplychainnode-) from beginning to end represents all entities who took custody of a given bid request.

> **Domain format note:** The `asi` and `domain` fields must be populated with only the canonical root domain of the advertising system or seller — the same domains used in OpenRTB `Site.domain`, sellers.json, and ads.txt. Full URLs and schemas (`http://`, `https://`) must not be used. The root domain is defined as the public suffix plus one label (e.g. `example.com`, `example.co.uk`).

### OpenRTB Supply Chain Object Implementation Rules

Version 1.0
- It is **invalid** for a reseller to copy the SupplyChain object from the previous seller without also inserting their own node into the chain. If a reseller doesn't insert themselves, their bid request should not include the SupplyChain object.
- If a seller is reselling inventory that **did not previously contain** a SupplyChain object, they should create the object themselves, set `complete` to `0`, and insert their node into the `nodes` array.
- If a seller is reselling inventory that **has** a SupplyChain object, the reseller should copy the existing object (preserving the original `complete` value) and append their node to the end of the `nodes` array.
- If this is the **originating bid request** for this inventory, the SupplyChain object should be created with `complete` set to `1` and only the originating node in the `nodes` array.
- It is invalid for a Seller ID to represent multiple entities. Every Seller ID must map to only a single entity that is paid for inventory transacted with that Seller ID. It is valid for a selling entity to have multiple Seller IDs within an advertising system.

Version 1.1
- All original rules continue to apply to updated versions
- hp=0 nodes reflect the order of the request, which may not be in the same order of the payment
- Where the same company is two hp=0 nodes in a row, collapse nodes. Where there are two nodes of the same company but one is hp=1 and the other is hp=0, both nodes should be enumerated. If both are hp=1, both should be enumerated. 
- Order reflects the outbound sequence, not the inbound sequence
- All third parties (e.g., not the end publisher) that take control of the bid request must be listed in the schain regardless of whether that third-party code is server-side or client-side.  
- Mediation layers (e.g. mobile) are treated the same as web for the purposes of this specification
- No ads.txt file will be required for hp=0 nodes, but an entry in the corresponding sellers.json will be strongly recommended.
- Sellers.json is strongly recommended for all nodes
- Version 1.1 and onward will use a new enumeration to denote that both paid and unpaid nodes are included. 


### Placement in the Bid Request

| OpenRTB Version | Location |
|---|---|
| 2.6+ | `BidRequest.source.schain` |
| 2.5 | `BidRequest.source.ext.schain` |
| 2.4 and prior | `BidRequest.ext.schain` |

### OpenRTB Examples - version 1.0

#### Valid, complete SupplyChain — single hop (originating bid request)

```json
{
  "id": "BidRequest1",
  "app": {
    "publisher": {
      "id": "00001"
    }
  },
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "directseller.com",
          "sid": "00001",
          "rid": "BidRequest1",
          "hp": 1
        }
      ]
    }
  }
}
```

#### Valid, complete SupplyChain — two hops (resale of above)

```json
{
  "id": "BidRequest2",
  "app": {
    "publisher": {
      "id": "aaaaa"
    }
  },
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "directseller.com",
          "sid": "00001",
          "rid": "BidRequest1",
          "hp": 1
        },
        {
          "asi": "reseller.com",
          "sid": "aaaaa",
          "rid": "BidRequest2",
          "hp": 1
        }
      ]
    }
  }
}
```

#### Valid, incomplete SupplyChain — upstream node does not support SupplyChain

```json
{
  "id": "BidRequest4",
  "app": {
    "publisher": {
      "id": "aaaaa"
    }
  },
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 0,
      "nodes": [
        {
          "asi": "reseller.com",
          "sid": "aaaaa",
          "rid": "BidRequest4",
          "hp": 1
        }
      ]
    }
  }
}
```

### OpenRTB Examples - version 1.1 
> **Notes on conventions used throughout:**
> - `hp=1` means the node is in the payment chain and paid directly by the downstream node.
> - `hp=0` means the node is not in the direct payment chain (e.g. SSAI vendor, passthrough, intermediary paid by fee).
> - `complete=1` means the full chain from origin to buyer is represented.
> - `complete=0` means the chain is incomplete — upstream nodes are unknown or omitted.
> - `sid` values are illustrative placeholders. In production these must match the seller_id in the named ASI's sellers.json.

### WEB-1: Device > Server-Side Header Bidding Wrapper > SSP > DSP

The server-side wrapper has a client-side JS component but processes and forwards server-side, hence hp=0.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "server-side-wrapper.com",
          "sid": "ssw-pub-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "ssw-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### server-side-wrapper.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@server-side-wrapper.com",
  "sellers": [
    {
      "seller_id": "ssw-pub-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "ssw-ssp-001",
      "name": "Server-Side Wrapper",
      "domain": "server-side-wrapper.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### WEB-2: Device > Open Bidding Provider > SSP > DSP

The open bidding provider sits between the device and the SSP, routing the request server-side. hp=0 because the SSP pays the open bidding provider, not the publisher directly.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "open-bidding-provider.com",
          "sid": "obp-pub-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "obp-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### open-bidding-provider.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@open-bidding-provider.com",
  "sellers": [
    {
      "seller_id": "obp-pub-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "obp-ssp-001",
      "name": "Open Bidding Provider",
      "domain": "open-bidding-provider.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### WEB-3: Device > Client-Side Wrapper > Server-Side Header Bidding Wrapper > SSP > DSP

Two wrapper layers. The client-side wrapper initiates from the browser (hp=0 — not in payment chain). The server-side wrapper receives and forwards (hp=0). The SSP runs the auction and pays upstream (hp=1).

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "client-side-wrapper.com",
          "sid": "csw-pub-001",
          "hp": 0
        },
        {
          "asi": "server-side-wrapper.com",
          "sid": "ssw-csw-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "ssw-ssp-002",
          "hp": 1
        }
      ]
    }
  }
}
```

#### client-side-wrapper.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@client-side-wrapper.com",
  "sellers": [
    {
      "seller_id": "csw-pub-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### server-side-wrapper.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@server-side-wrapper.com",
  "sellers": [
    {
      "seller_id": "ssw-csw-001",
      "name": "Client-Side Wrapper",
      "domain": "client-side-wrapper.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "ssw-ssp-002",
      "name": "Server-Side Wrapper",
      "domain": "server-side-wrapper.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### WEB-4: Device > Prebid JS > SSP > DSP

The publisher's own domain initiates the Prebid request directly. Publisher is named as node 1 because they are directly initiating — no intermediary wrapper sits above them.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "publishersite.com",
          "sid": "pub-direct-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "pub-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### publishersite.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@publishersite.com",
  "sellers": [
    {
      "seller_id": "pub-direct-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "pub-ssp-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

---

### WEB-5: Device > Publisher Ad Management JS > Publisher Ad Management Server > SSP > DSP

The publisher ad management platform operates both the client-side JS and a server-side component. Both are operated by the same entity, so only one node is needed. hp=1 because the platform is in the payment chain — the SSP pays them, they pay the publisher.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "publisher-ad-management-platform.com",
          "sid": "pamp-pub-001",
          "hp": 1
        },
        {
          "asi": "ssp.com",
          "sid": "pamp-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### publisher-ad-management-platform.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@publisher-ad-management-platform.com",
  "sellers": [
    {
      "seller_id": "pamp-pub-001",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "pamp-ssp-001",
      "name": "Publisher Ad Management Platform",
      "domain": "publisher-ad-management-platform.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### WEB-6: Device > Publisher Ad Management JS > Yield Optimization Intermediary > SSP > DSP

The publisher ad management platform's JS initiates the request (hp=1 — in payment chain). The yield optimization intermediary sits between it and the SSP but does not participate in payment (hp=0). The SSP pays the publisher ad management platform directly, which pays the publisher.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "publisher-ad-management-platform.com",
          "sid": "pamp-pub-002",
          "hp": 1
        },
        {
          "asi": "yield-optimization-intermediary.com",
          "sid": "yoi-pamp-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "yoi-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### publisher-ad-management-platform.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@publisher-ad-management-platform.com",
  "sellers": [
    {
      "seller_id": "pamp-pub-002",
      "name": "Example Publisher LLC",
      "domain": "publishersite.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### yield-optimization-intermediary.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@yield-optimization-intermediary.com",
  "sellers": [
    {
      "seller_id": "yoi-pamp-001",
      "name": "Publisher Ad Management Platform",
      "domain": "publisher-ad-management-platform.com",
      "seller_type": "INTERMEDIARY",
      "is_passthrough": 1
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "yoi-ssp-001",
      "name": "Publisher Ad Management Platform (via Yield Optimization Intermediary)",
      "domain": "publisher-ad-management-platform.com",
      "seller_type": "INTERMEDIARY",
      "custodian": "yield-optimization-intermediary.com"
    }
  ]
}
```

---

## Mobile Examples

---

### MOB-1: Client Device > Mobile SDK > DSP

The mobile SDK does two jobs: client-side connection and initial payload construction. It sends directly to the DSP. Single node, hp=1 because it is in the payment chain.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "mobile-sdk.com",
          "sid": "msdk-pub-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### mobile-sdk.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@mobile-sdk.com",
  "sellers": [
    {
      "seller_id": "msdk-pub-001",
      "name": "Example App Developer LLC",
      "domain": "appdeveloper.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

---

### MOB-2: Client Device > Mobile SDK > SSP > DSP (Classic resell)

Classic reselling. DSP pays SSP. SSP pays Mobile SDK. Mobile SDK pays the app developer. No hp=0 nodes — every node is in the payment chain.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "mobile-sdk.com",
          "sid": "msdk-pub-002",
          "hp": 1
        },
        {
          "asi": "ssp.com",
          "sid": "msdk-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### mobile-sdk.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@mobile-sdk.com",
  "sellers": [
    {
      "seller_id": "msdk-pub-002",
      "name": "Example App Developer LLC",
      "domain": "appdeveloper.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "msdk-ssp-001",
      "name": "Mobile SDK",
      "domain": "mobile-sdk.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### MOB-3: Client Device > Mobile SDK > SSP > DSP (SDK Renting)

SDK renting: the SSP pays the publisher directly, bypassing the SDK in the payment chain. Mobile SDK is hp=0 — it facilitates the request but is not paid for media by the SSP.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "mobile-sdk.com",
          "sid": "msdk-pub-003",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "msdk-ssp-002",
          "hp": 1
        }
      ]
    }
  }
}
```

#### mobile-sdk.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@mobile-sdk.com",
  "sellers": [
    {
      "seller_id": "msdk-pub-003",
      "name": "Example App Developer LLC",
      "domain": "appdeveloper.com",
      "seller_type": "PUBLISHER",
      "is_passthrough": 1,
      "comment": "SSP pays publisher directly. Must establish account relationship with publisher to transact."
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "msdk-ssp-002",
      "name": "Example App Developer LLC",
      "domain": "appdeveloper.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

---

## CTV Examples

---

### CTV-1: Device > SSAI Platform > CTV Ad Server > SSP > DSP (Canonical single path)

SSAI platform stitches ads server-side (hp=0). CTV ad server manages the auction request (hp=0). SSP runs the auction and pays the publisher (hp=1).

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-platform.com",
          "sid": "ssai-pub-001",
          "hp": 0
        },
        {
          "asi": "ctv-ad-server.com",
          "sid": "cas-ssai-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "cas-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### ssai-platform.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssai-platform.com",
  "sellers": [
    {
      "seller_id": "ssai-pub-001",
      "name": "Example CTV Publisher LLC",
      "domain": "ctvpublisher.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ctv-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ctv-ad-server.com",
  "sellers": [
    {
      "seller_id": "cas-ssai-001",
      "name": "SSAI Platform",
      "domain": "ssai-platform.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "cas-ssp-001",
      "name": "CTV Ad Server",
      "domain": "ctv-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-2: Device > Pure SSAI Vendor > CTV Ad Server > SSP > DSP

Same shape as CTV-1. Confirms the framework treats single-purpose SSAI vendors the same as cloud-platform SSAIs.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "pure-ssai-vendor.com",
          "sid": "pssai-pub-001",
          "hp": 0
        },
        {
          "asi": "ctv-ad-server.com",
          "sid": "cas-pssai-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "cas-ssp-002",
          "hp": 1
        }
      ]
    }
  }
}
```

#### pure-ssai-vendor.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@pure-ssai-vendor.com",
  "sellers": [
    {
      "seller_id": "pssai-pub-001",
      "name": "Example CTV Publisher LLC",
      "domain": "ctvpublisher.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ctv-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ctv-ad-server.com",
  "sellers": [
    {
      "seller_id": "cas-pssai-001",
      "name": "Pure SSAI Vendor",
      "domain": "pure-ssai-vendor.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "cas-ssp-002",
      "name": "CTV Ad Server",
      "domain": "ctv-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-3: Device > SSAI + Ad Server (Same Operator) > SSP > DSP

SSAI and ad server are operated by the same company. They collapse into a single node. hp=0 because they do not pay the publisher — the SSP does.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-and-ad-server-operator.com",
          "sid": "saso-pub-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "saso-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### ssai-and-ad-server-operator.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssai-and-ad-server-operator.com",
  "sellers": [
    {
      "seller_id": "saso-pub-001",
      "name": "Example CTV Publisher LLC",
      "domain": "ctvpublisher.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "saso-ssp-001",
      "name": "SSAI and Ad Server Operator",
      "domain": "ssai-and-ad-server-operator.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-4: Device > Publisher-Owned SSAI > Primary Ad Server > Content Owner Ad Server > SSP > DSP

Publisher-owned SSAI initiates (hp=0). Primary ad server manages the request (hp=0). Content owner ad server is called to honor right of first refusal on content owner inventory (hp=0). SSP pays the publisher (hp=1).

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "publisher-owned-ssai.com",
          "sid": "possai-pub-001",
          "hp": 0
        },
        {
          "asi": "primary-ad-server.com",
          "sid": "pas-possai-001",
          "hp": 0
        },
        {
          "asi": "content-owner-ad-server.com",
          "sid": "coas-pas-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "coas-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### publisher-owned-ssai.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@publisher-owned-ssai.com",
  "sellers": [
    {
      "seller_id": "possai-pub-001",
      "name": "Example App Owner LLC",
      "domain": "appowner.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### primary-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@primary-ad-server.com",
  "sellers": [
    {
      "seller_id": "pas-possai-001",
      "name": "Publisher-Owned SSAI",
      "domain": "publisher-owned-ssai.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### content-owner-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@content-owner-ad-server.com",
  "sellers": [
    {
      "seller_id": "coas-pas-001",
      "name": "Primary Ad Server",
      "domain": "primary-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "coas-ssp-001",
      "name": "Content Owner Ad Server",
      "domain": "content-owner-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-5a: Device > SSAI > App Owner Ad Server > SSP > DSP (App Owner path — inventory share)

One of two parallel bid requests generated from the same ad break. This path represents the app owner's inventory share. The originating SSAI is the same in both CTV-5a and CTV-5b, but the ad server and seller differ.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-vendor.com",
          "sid": "ssaiv-appowner-001",
          "hp": 0
        },
        {
          "asi": "app-owner-ad-server.com",
          "sid": "aoas-ssaiv-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "aoas-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### ssai-vendor.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssai-vendor.com",
  "sellers": [
    {
      "seller_id": "ssaiv-appowner-001",
      "name": "Example App Owner LLC",
      "domain": "appowner.com",
      "seller_type": "PUBLISHER"
    },
    {
      "seller_id": "ssaiv-invpartner-001",
      "name": "Inventory Share Partner LLC",
      "domain": "inventorypartner.com",
      "seller_type": "PUBLISHER",
      "comment": "Inventory share partner path — see CTV-5b"
    }
  ]
}
```

#### app-owner-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@app-owner-ad-server.com",
  "sellers": [
    {
      "seller_id": "aoas-ssaiv-001",
      "name": "SSAI Vendor",
      "domain": "ssai-vendor.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "aoas-ssp-001",
      "name": "App Owner Ad Server",
      "domain": "app-owner-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-5b: Device > SSAI > Inventory Share Partner Ad Server > SSP > DSP (Inventory share partner path)

Second of two parallel bid requests from the same ad break. Same SSAI originator as CTV-5a, different ad server and seller — representing the inventory share partner's portion of the pod.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-vendor.com",
          "sid": "ssaiv-invpartner-001",
          "hp": 0
        },
        {
          "asi": "inventory-share-partner-ad-server.com",
          "sid": "ispas-ssaiv-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "ispas-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### inventory-share-partner-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@inventory-share-partner-ad-server.com",
  "sellers": [
    {
      "seller_id": "ispas-ssaiv-001",
      "name": "SSAI Vendor",
      "domain": "ssai-vendor.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json (addendum for CTV-5b)

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "ispas-ssp-001",
      "name": "Inventory Share Partner Ad Server",
      "domain": "inventory-share-partner-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-6: Device > SSAI Platform > Content Owner Ad Server > Primary CTV Ad Server > SSP > DSP

Three custodians precede the SSP. SSAI platform generates the initial payload (hp=0). Content owner ad server holds custody next (hp=0). Primary CTV ad server (the INVENTORYPARTNERDOMAIN entity) then initiates the SSP auction (hp=0). SSP pays the publisher (hp=1).

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-platform.com",
          "sid": "ssaip-pub-002",
          "hp": 0
        },
        {
          "asi": "content-owner-ad-server.com",
          "sid": "coas-ssaip-001",
          "hp": 0
        },
        {
          "asi": "primary-ctv-ad-server.com",
          "sid": "pcas-coas-001",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "pcas-ssp-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### ssai-platform.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssai-platform.com",
  "sellers": [
    {
      "seller_id": "ssaip-pub-002",
      "name": "Example CTV Publisher LLC",
      "domain": "ctvpublisher.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### content-owner-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@content-owner-ad-server.com",
  "sellers": [
    {
      "seller_id": "coas-ssaip-001",
      "name": "SSAI Platform",
      "domain": "ssai-platform.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### primary-ctv-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@primary-ctv-ad-server.com",
  "sellers": [
    {
      "seller_id": "pcas-coas-001",
      "name": "Content Owner Ad Server",
      "domain": "content-owner-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp.com",
  "sellers": [
    {
      "seller_id": "pcas-ssp-001",
      "name": "Primary CTV Ad Server",
      "domain": "primary-ctv-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

---

### CTV-7: Device > SSAI Platform > Content Owner Ad Server > Primary CTV Ad Server > SSP > DSP (Sequential Ad Server Connections)

Identical node shape to CTV-6. The distinction is architectural: the primary ad server forwards to a secondary ad server to access differentiated demand in a single sequential chain, rather than parallel fan-out. One bid request, both ad servers as sequential nodes in a single schain.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-platform.com",
          "sid": "ssaip-pub-003",
          "hp": 0
        },
        {
          "asi": "content-owner-ad-server.com",
          "sid": "coas-ssaip-002",
          "hp": 0
        },
        {
          "asi": "primary-ctv-ad-server.com",
          "sid": "pcas-coas-002",
          "hp": 0
        },
        {
          "asi": "ssp.com",
          "sid": "pcas-ssp-002",
          "hp": 1
        }
      ]
    }
  }
}
```

> sellers.json entries for CTV-7 follow the same structure as CTV-6 with updated sid references. Each ASI's sellers.json lists the entity upstream of it as either PUBLISHER or INTERMEDIARY per the same logic.

---

### CTV-8: Device > SSAI Platform > Primary CTV Ad Server > SSP-1 > SSP-2 > DSP (SSP Chaining)

SSP-1 routes to SSP-2 for additional bid density or deal access. Both SSPs are hp=1 — both are in the payment chain. SSP-2 pays SSP-1 pays the publisher.

#### OpenRTB 2.6 schain object

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "ssai-platform.com",
          "sid": "ssaip-pub-004",
          "hp": 0
        },
        {
          "asi": "primary-ctv-ad-server.com",
          "sid": "pcas-ssaip-001",
          "hp": 0
        },
        {
          "asi": "ssp-1.com",
          "sid": "ssp1-pcas-001",
          "hp": 1
        },
        {
          "asi": "ssp-2.com",
          "sid": "ssp2-ssp1-001",
          "hp": 1
        }
      ]
    }
  }
}
```

#### ssai-platform.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssai-platform.com",
  "sellers": [
    {
      "seller_id": "ssaip-pub-004",
      "name": "Example CTV Publisher LLC",
      "domain": "ctvpublisher.com",
      "seller_type": "PUBLISHER"
    }
  ]
}
```

#### primary-ctv-ad-server.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@primary-ctv-ad-server.com",
  "sellers": [
    {
      "seller_id": "pcas-ssaip-001",
      "name": "SSAI Platform",
      "domain": "ssai-platform.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp-1.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp-1.com",
  "sellers": [
    {
      "seller_id": "ssp1-pcas-001",
      "name": "Primary CTV Ad Server",
      "domain": "primary-ctv-ad-server.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```

#### ssp-2.com/sellers.json

```json
{
  "version": "1.0",
  "contact_email": "adops@ssp-2.com",
  "sellers": [
    {
      "seller_id": "ssp2-ssp1-001",
      "name": "SSP-1",
      "domain": "ssp-1.com",
      "seller_type": "INTERMEDIARY"
    }
  ]
}
```



### 2.6 Non-OpenRTB Tag Serialization

When SupplyChain information must be passed via an ad tag or VAST URL rather than an OpenRTB request, a string serialization format is used.

**Format:**

```
{ver},{complete}!{asi},{sid},{hp},{rid},{name},{domain}!{asi},{sid},{hp},{rid},{name},{domain}...
```

The SupplyChain object properties (`ver` and `complete`) appear first, separated by a comma. Each node follows, separated from the header and from each other by a bang (`!`). Node properties are comma-separated in the order: `asi, sid, hp, rid, name, domain, ext`. Optional trailing properties may be omitted.

If any property value contains a comma or bang character, it must be URL encoded per RFC 3986.

**Recommended URL parameter name:** `schain`

**Behavior rules:**
- If receiving a tag-based request and sending an outbound OpenRTB request: parse the serialized string, populate the SupplyChain object, append your own node.
- If receiving a tag-based request and sending an outbound tag: append your node to the pre-existing string without altering any preceding information.
- If receiving an OpenRTB request and sending an outbound tag: serialize the received SupplyChain object, then append your own node.

**Serialization examples:**

| Scenario | Serialized Value |
|---|---|
| Single hop, complete | `1.0,1!exchange1.com,1234,1,bid-request-1,publisher,publisher.com` |
| Single hop, complete, optional fields omitted | `1.0,1!exchange1.com,1234,1,,,` |
| Multiple hops, complete | `1.0,1!exchange1.com,1234,1,,,!exchange2.com,abcd,1,,,` |
| Multiple hops, incomplete | `1.0,0!exchange2.com,abcd,1,,,` |
| Encoded values (sid contains comma) | `1.0,1!exchange1.com,1234%21abcd,1,bid-request-1,publisher%2c%20Inc.,publisher.com` |

---

## 3. Inventory Sharing: Ads.txt & App-ads.txt Explainer

### 3.1 Background

The Connected TV market introduces a much higher occurrence of complex monetization relationships that make ads.txt & app-ads.txt, as currently designed, insufficient for broader adoption. These guidelines and spec enhancements are intended to more seamlessly support sites and apps in which multiple entities may have ownership rights over the ad space — commonly referred to as **inventory sharing**.

In OTT, these situations often arise from content distribution relationships (MVPDs or vMVPDs) or various carriage agreements (TV Everywhere). Nearly all such relationships can be simplified to: *some business entity, other than the app/site owner, has ownership over some ad space within the app/site and the right to sell that inventory.*

In the previous version of ads.txt & app-ads.txt, declaring this relationship would require the app/site owner to maintain their file with the publisher IDs of all partners (and their resellers) with whom they have negotiated inventory ownership rights — making many files prohibitively difficult to maintain. The updated standard instead allows the app/site to point to a partner domain whose ads.txt or app-ads.txt file is authoritative for that partner's inventory.

### 3.2 Scope

**In scope:**
- **Inventory sharing** relationships, where different entities (app/site owner, content owner) may have the right to sell ad space within a piece of content on a given app/site.
- Addressing basic misrepresentations of publishers' ownership of (and rights to sell) inventory on an app/site.

**Out of scope:**
- **Revenue sharing** scenarios where one entity owns and sells all inventory and shares proceeds with content owners (e.g. YouTube model). Only one entity owns/sells the inventory; this is not inventory sharing.
- Content rights ownership, exclusivity, or inventory ownership within specific shows or programming.

> **MANAGERDOMAIN vs. INVENTORYPARTNERDOMAIN:** `MANAGERDOMAIN` represents a primary or exclusive programmatic seller of a publisher's inventory and is the most direct path to that inventory. Managers participate in the transaction as revenue-share intermediaries and are expected to appear in the schain as the terminal node. `INVENTORYPARTNERDOMAIN` represents a company that owns or has the right to sell a portion of ads on the underlying app — they are the final payee and receive all proceeds from their share of inventory. `INVENTORYPARTNERDOMAIN` is primarily intended for CTV/OTT inventory sharing.

### 3.3 Updates to the Standard

The ads.txt & app-ads.txt specs have been updated to include the ability to designate another domain that can validate the publisher ID of the bid request. These domains are passed in the OpenRTB bid request via:

- `app.inventorypartnerdomain`
- `site.inventorypartnerdomain`

The corresponding declaration in the app/site owner's ads.txt or app-ads.txt file:

| Variable | Value | Description |
|---|---|---|
| `inventorypartnerdomain` | Domain of the partner with ownership of some inventory on this site/app | When a site or app contains ad inventory owned by another partner, list all such partner domains via this directive. The partner's ads.txt or app-ads.txt will be hosted at the declared domain. |

### 3.4 Example Use Cases

**Definitions:**
- **Programmer A** — Content owner or content developer (e.g. broadcaster, streaming studio)
- **(v)MVPD** — (Virtual) Multichannel Video Programming Distributor / content distributor

#### Case A: Programmer-Owned App

All inventory is produced and sold by Programmer A in their own app via their chosen SSPs. This does not differ from the current app-ads.txt authorization model.

**OpenRTB declaration (by Programmer A):**
```json
{
  "app": {
    "bundle": "12345",
    "storeurl": "https://ctvappstore.com/details/12345/programmerA",
    "publisher": {
      "id": "abcde"
    }
  }
}
```

**Programmer A app-ads.txt** (`devsite.programmerA.com/app-ads.txt`):
```
ssp.com, abcde, DIRECT
```

#### Case B: Content Channel on vMVPD App

Two business scenarios arise depending on who has the right to sell the ad slot.

**Scenario B.1 — vMVPD sells the slot (content ownership blinded):**

This does not differ from the current app-ads.txt model.

**OpenRTB declaration (by vMVPD B):**
```json
{
  "app": {
    "bundle": "67890",
    "storeurl": "https://ctvappstore.com/details/67890/vmvpdB",
    "publisher": {
      "id": "vwxyz"
    }
  }
}
```

**vMVPD B app-ads.txt** (`devsite.vMVPDB.com/app-ads.txt`):
```
ssp.com, vwxyz, DIRECT
```

**Scenario B.2 — Programmer A sells the slot within the vMVPD app:**

**OpenRTB declaration (by Programmer A):**
```json
{
  "app": {
    "bundle": "67890",
    "storeurl": "https://ctvappstore.com/details/67890/vmvpdB",
    "inventorypartnerdomain": "programmerA.com",
    "publisher": {
      "id": "abcde"
    }
  }
}
```

**vMVPD B app-ads.txt** (`devsite.vMVPDB.com/app-ads.txt`):
```
ssp.com, vwxyz, DIRECT
inventorypartnerdomain=programmerA.com
```

**Programmer A ads.txt** (`programmerA.com/ads.txt`):
```
ssp.com, abcde, DIRECT
```

#### Case C: Programmer-Owned App with MVPD Sign-In (TV Everywhere)

Due to user login with vMVPD B's credentials in Programmer A's app, vMVPD B has the right to sell the ad slot. This is the reverse of Case B.2.

**OpenRTB declaration (by vMVPD B):**
```json
{
  "app": {
    "bundle": "12345",
    "storeurl": "https://ctvappstore.com/details/12345/programmerA",
    "inventorypartnerdomain": "vmvpdB.com",
    "publisher": {
      "id": "vwxyz"
    }
  }
}
```

**Programmer A app-ads.txt** (`devsite.programmerA.com/app-ads.txt`):
```
ssp.com, abcde, DIRECT
inventorypartnerdomain=vmvpdB.com
```

**vMVPD B ads.txt** (`vMVPDB.com/ads.txt`):
```
ssp.com, vwxyz, DIRECT
```

### 3.5 Use Case Implementation Logic

| Scenario | Action | Possible Outcomes |
|---|---|---|
| Bid request is app and **does not have** `app.inventorypartnerdomain` (Cases A, B.1) | Look up app-ads.txt at developer domain from app store record for `app.bundle` | No valid app-ads.txt found → **Nonparticipating**; Publisher ID + ad system not in file → **Unauthorized**; Publisher ID + ad system found → **Authorized** |
| Bid request is app and **has** `app.inventorypartnerdomain` (Cases B.2, C) | Look up app-ads.txt at developer domain AND ads.txt at `app.inventorypartnerdomain` | No app-ads.txt found → **Inconclusive**; Publisher ID found directly in app-ads.txt → **Authorized**; app-ads.txt found but no matching `inventorypartnerdomain` directive → **Unauthorized**; Matching directive found but no ads.txt at that domain → **Inconclusive**; Publisher ID not in partner's ads.txt → **Unauthorized**; Publisher ID found in partner's ads.txt → **Authorized** |

*For site bid requests, substitute `site.inventorypartnerdomain` for `app.inventorypartnerdomain`.*

### 3.6 Implementation Guidelines for CTV/OTT

#### CTV App Store Requirements

1. Make the app store website publicly available on the web.
2. Support the [OTT/CTV Store Assigned App Identification Guidelines](https://iabtechlab.com/wp-content/uploads/2020/08/IAB-Tech-Lab-OTT-store-assigned-App-Identification-Guidelines-2020.pdf) — store-assigned IDs must be publicly accessible.
3. Follow the app-ads.txt standard — add meta tags to HTML pages publishing the developer website URL, bundle ID, and store ID.

#### Publisher Requirements

1. Adopt the OTT/CTV Store Assigned App Identification Guidelines:
   - Pass CTV app store assigned IDs in `app.bundle` (OpenRTB 2.5) or `app.storeid` (OpenRTB 3.0/AdCOM 1.0).
   - Pass store URL in `app.storeurl`.
2. App owners should publish their app-ads.txt on their developer website.
3. App owners should declare `inventorypartnerdomain` for any partners with ownership of inventory within their app.

**Guidance on additional contextual signals that should not be stuffed into bundle fields:**

| Signal | Correct Field |
|---|---|
| Channel or network within app | `content.producer.name` |
| VOD vs. livestream | `content.livestream` |
| Device make/model | `device.make` and `device.model` |
| App store | `app.storeurl` |

#### SSP/Exchange Requirements

1. Implement `app.inventorypartnerdomain` and `site.inventorypartnerdomain` to support inventory partner domain pointers.
2. Support publishers in providing CTV App IDs per the OTT/CTV Store Assigned App Identification Guidelines.

#### DSP Requirements

1. Implement the app-ads.txt crawler according to standardized guidance from CTV app stores (or use a compliant service).

---

## 4. FAQ: sellers.json and SupplyChain Object

*Source: IAB Tech Lab FAQ for sellers.json and SupplyChain Object (July 2019, updated September 2020)*

### 4.1 What Are sellers.json and SupplyChain?

sellers.json and SupplyChain are the mechanisms to identify all intermediaries that participate in the flow of money from the buying platform back to the publisher. They do not include any intermediary that does not participate in money flow — systems paid a flat fee for their services but that do not pay upstream sellers are not included. In cases of complicated supply chains, this enables increased transparency and the ability to identify and prevent fraudulent or otherwise unacceptable supply sources, according to the business policies of the consuming advertising system.

### 4.2 VAST and Tag-Based Requests

If the request is for inventory sold on behalf of the publisher, the ad system provides the SupplyChain information and the tag does not need to contain SupplyChain information. If the request is for inventory sold on behalf of an intermediary, the SupplyChain node for the intermediary (and any upstream intermediaries) are provided in the tag, and the SupplyChain node for the ad system is appended prior to sending the request.

Advertising systems should support receiving supply chain details from the SupplyChain object as specified in the [serialization format](#26-non-openrtb-tag-serialization) above.

### 4.3 SSAI Vendors

All intermediaries that are part of the chain of payments — ranging from the buying system to the publisher — are expected to be included in the SupplyChain. If an SSAI vendor is acting as an intermediary in the payment chain, they should be included. If an SSAI vendor is acting purely as an ad serving vendor — paid an ad serving fee by the publisher and not involved in the media money flow — they would not be listed.

### 4.4 Correctness and Fraud

The presence of a sellers.json or SupplyChain object does **not** guarantee correctness. These are tools to provide additional information that should be verified by consumers (i.e. DSPs). Consumers should expect and defend against potentially falsified information.

### 4.5 Seller ID

A seller ID is a unique ID assigned by an advertising system to each of the inventory sellers on their platform. It is usually conveyed in the `id` field of the `publisher` object in an OpenRTB bid request. It is also the same ID as found in column two of ads.txt files.

For the purposes of sellers.json and SupplyChain, the seller ID must represent a single entity that the advertising system pays directly for inventory. Multiple seller IDs may be used to represent a single business on one advertising system, but one seller ID cannot represent multiple businesses.

### 4.6 Validation Methods

#### Validating SupplyChain 1.0 information

- For a given payment handling node (hp=1), the name associated with a seller ID (from sellers.json) on a given advertising system should match the advertising system in the preceding hp=1 node. Otherwise, it implies a break in the chain.
- Payment handling nodes require corresponding ads.txt records for a given domain/app, and should be present for upstream nodes in the SupplyChain for that domain/app. Note that this is expanded guidance from the existing ads.txt spec, but should be considered a best practice.
- DSPs could do spot checks and ask publishers if a supply chain looks valid with how the publisher expects their inventory is sold. They can also use SupplyChain information to inform the total inventory sold via a particular chain or intermediary for any arbitrary length of time.
- In cases where the ‘complete’ attribute is set to 1 (payment complete), you can check that the entity name for the first node is consistent with the known owner of the site or app.
- When `complete=1`, check that the entity name for the first node is consistent with the known owner of the site or app.
- When `complete=1`, check that the first node has a `seller_type` of PUBLISHER. If it does not, there must be one or more missing nodes.
- Check that the first node is listed as a DIRECT seller in the publisher's ads.txt. If it is not, either the actual first node has been removed (chain has been tampered with), or the publisher has incorrectly listed the record as RESELLER.

#### Supporting SupplyChain 1.1
- Supply Chain version 1.1 and above, additional sellers.json entries for non-payment handling entities are available. 
- Existing sellers.json entries don’t need to have their details changed, new fields should be added as needed.
- The order of the Supply Chain Nodes for non-payment handling entities should in the order of the bid request. Implementers should expect to see one or more hp=0 nodes in front of the first payment handling node. hp=0 nodes after the first payment handling entity will also not be uncommon.  
- DSP schain validation logic should skip hp=0 nodes; but should instead look for is_passthrough=1 and a corresponding entry in the sellers.json file of the appropriate node.

#### Validating sellers.json information

- DSPs can spot check by asking publishers to confirm when a sellers.json file claims a specific seller ID represents that publisher directly.
- Look for irregular patterns: a 1:1 correlation between publisher ID and domain/app across the board suggests incorrect use of seller ID. Multiple apparently unrelated apps/domains observed for a single seller ID with `seller_type` set to PUBLISHER is also suspicious.
- General consistency should be observed between DIRECT seller accounts in a site's ads.txt and the `seller_type`, `is_passthrough`, and entity name found in sellers.json for a given advertising system.

### 4.7 Publisher Responsibilities

**Publishers themselves do not need to implement sellers.json or SupplyChain.** These specifications are implemented by or consumed by advertising systems: DSPs, SSPs/exchanges, and ad servers. Similarly, there is no need for the publisher to supply a SupplyChain object via tags that are not sent on behalf of intermediaries.

### 4.8 Setting seller_type

Use the following logic to determine the correct `seller_type` value for a given seller in your sellers.json:

```
Does your advertising system pay the named entity directly for the inventory?
├── No  → INTERMEDIARY
└── Yes → Does the named entity own the site or app containing the inventory?
          ├── No  → INTERMEDIARY
          └── Yes → PUBLISHER
```

If some combination of Yes and No answers apply across different inventory for the same seller, set `BOTH`.

### 4.9 Setting is_passthrough

`is_passthrough=1` is set when an advertising system requires the **consuming system** (the entity receiving the bid request) to hold a direct contractual relationship with the named entity. The consuming advertising system may further broadcast the bid request to others, but does not set `is_passthrough=1` in its sellers.json unless it also requires the receiver to hold a direct contractual relationship with the supply source.

```
Do you require the system receiving your bid requests
to hold a contract directly with this entity?
├── Yes → is_passthrough = 1
└── No  → is_passthrough = 0
```

**Important:** Only the advertising system that is acting as a passthrough should set `is_passthrough=1`. Publishers and intermediaries that *supply inventory to* a passthrough system do not set this field.

### 4.10 Header Bidding

Technology vendors that are not in the direct payment chain between the buying system and the publisher should not be listed in the SupplyChain object. There is also no need to assign a seller ID to these vendors.

### 4.11 Worked Examples

The following examples show how sellers.json, SupplyChain, and ads.txt work together across common real-world scenarios.

---

#### Standard Header Bidding or Tag — Publisher Direct

A publisher sends an ad request directly to an exchange. The exchange runs the auction on behalf of the publisher. The schain contains a single node.

**exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "184003",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com"
    }
  ]
}
```

**SupplyChain object in OpenRTB bid request from exchange.com:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "exchange.com",
          "sid": "184003",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Standard Header Bidding or Tag — Ad Network

A publisher authorizes an ad network to sell their inventory. The ad network transmits a bid request to an exchange that already contains SupplyChain info. The exchange appends its own node.

**ads.txt entry on publisher site:**
```
exchange.com, <adnetwork_id>, RESELLER
```

**adnetwork.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "1200",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com"
    }
  ]
}
```

**exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "184033B",
      "name": "Ad Network",
      "seller_type": "INTERMEDIARY",
      "domain": "adnetwork.com"
    }
  ]
}
```

**SupplyChain object in OpenRTB bid request from exchange.com:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "adnetwork.com",
          "sid": "1200",
          "hp": 1
        },
        {
          "asi": "exchange.com",
          "sid": "184033B",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Passthrough — Publisher to Exchange Bidding Participant

Inventory comes from the publisher. The Exchange Bidding participant pays the Exchange Bidding provider. The Exchange Bidding provider pays the publisher. The Exchange Bidding provider sets `is_passthrough=1` for the publisher entry, signaling that the receiving system must establish a direct account relationship with the publisher.

**ads.txt:** `DIRECT`

**eb-exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "pub-0978064532142215",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com",
      "is_passthrough": 1,
      "comment": "Must establish account relationship with publisher to transact"
    }
  ]
}
```

**eb-participant.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "184044",
      "name": "EB Exchange",
      "seller_type": "INTERMEDIARY",
      "domain": "eb-exchange.com",
      "comment": "Publisher via Exchange Bidding"
    }
  ]
}
```

**SupplyChain object in OpenRTB bid request from eb-participant.com:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "eb-exchange.com",
          "sid": "pub-0978064532142215",
          "hp": 1
        },
        {
          "asi": "eb-participant.com",
          "sid": "184044",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Passthrough — Reseller to Exchange Bidding Participant

Inventory comes from a reseller. The Exchange Bidding participant pays the Exchange Bidding provider. The Exchange Bidding provider pays the reseller, who pays the publisher. The Exchange Bidding provider sets `is_passthrough=1` for the reseller entry.

**ads.txt:** `RESELLER`

**reseller-exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "215",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com"
    }
  ]
}
```

**eb-exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "pub-3153065230153281",
      "name": "Reseller Exchange",
      "seller_type": "INTERMEDIARY",
      "domain": "reseller-exchange.com",
      "is_passthrough": 1,
      "comment": "Must establish account relationship with Reseller Exchange to transact"
    }
  ]
}
```

**eb-participant.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "185176",
      "name": "EB Exchange",
      "seller_type": "INTERMEDIARY",
      "domain": "eb-exchange.com",
      "comment": "Reseller Exchange via Exchange Bidding"
    }
  ]
}
```

**SupplyChain object in OpenRTB bid request from eb-participant.com:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "reseller-exchange.com",
          "sid": "215",
          "hp": 1
        },
        {
          "asi": "eb-exchange.com",
          "sid": "pub-3153065230153281",
          "hp": 1
        },
        {
          "asi": "eb-participant.com",
          "sid": "185176",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Passthrough — Exchange to Buyer via Passthrough Exchange

Buyer pays the passthrough exchange. Passthrough exchange pays Exchange 1. Exchange 1 and the buyer have an account control relationship. The passthrough exchange sets `is_passthrough=1` for Exchange 1's entries.

**ads.txt:** Same status as if passthrough exchange were not involved.

**exchange-1.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "2000",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com"
    },
    {
      "seller_id": "2001",
      "name": "Intermediary Exchange",
      "seller_type": "INTERMEDIARY",
      "domain": "intermediary-exchange.com"
    }
  ]
}
```

**passthrough-exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "exchange1_2000",
      "name": "Exchange 1",
      "seller_type": "INTERMEDIARY",
      "domain": "exchange-1.com",
      "is_passthrough": 1,
      "comment": "Must establish account with Exchange 1 to transact"
    },
    {
      "seller_id": "exchange1_2001",
      "name": "Exchange 1",
      "seller_type": "INTERMEDIARY",
      "domain": "exchange-1.com",
      "is_passthrough": 1,
      "comment": "Must establish account with Exchange 1 to transact"
    }
  ]
}
```

**SupplyChain — path starting from publisher:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "exchange-1.com",
          "sid": "2000",
          "hp": 1
        },
        {
          "asi": "passthrough-exchange.com",
          "sid": "exchange1_2000",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain — path starting from intermediary exchange:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "intermediary-exchange.com",
          "sid": "ABC",
          "hp": 1
        },
        {
          "asi": "exchange-1.com",
          "sid": "2001",
          "hp": 1
        },
        {
          "asi": "passthrough-exchange.com",
          "sid": "exchange1_2001",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Passthrough — Exchange Bidding Participant Pays Publisher Directly

The Exchange Bidding participant pays the publisher directly. No intermediary sits in the payment chain between them.

**ads.txt:** `DIRECT`

**eb-participant.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "184044-B",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com",
      "comment": "via Exchange Bidding TAM-style arrangement"
    }
  ]
}
```

**SupplyChain object in OpenRTB bid request from eb-participant.com:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "eb-participant.com",
          "sid": "184044-B",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Sales House Scenario

A sales house / rep firm does not own or operate any ad tech platform. It acts as monetization manager for other publishers' inventory and has its own O&O site. It sells all inventory programmatically through a single exchange account.

The sales house has three types of arrangements:
1. A publisher gives them 100% of inventory to manage and sells nothing directly.
2. A publisher-network gives them all of their inventory in one geography to manage; the network also monetizes sites it does not own.
3. The sales house has its own O&O site.

**Key principle:** SupplyChain represents money flows. The first node (when `complete=1`) is the first entity downstream from the actual site owner. The sales house's sellers.json represents each upstream business entity it pays. No entry is needed for O&O inventory since the sales house does not receive that inventory from anyone else.

**sales-house.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "1",
      "name": "Managed Publisher",
      "seller_type": "PUBLISHER",
      "domain": "managedpublisher.com"
    },
    {
      "seller_id": "2",
      "name": "Publisher Network",
      "seller_type": "BOTH",
      "domain": "publishernetwork.com"
    }
  ]
}
```

**SupplyChain sent by sales house to exchange — for managed publisher inventory:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "sales-house.com",
          "sid": "1",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain sent from exchange to buying system — for managed publisher inventory:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "sales-house.com",
          "sid": "1",
          "hp": 1
        },
        {
          "asi": "exchange.com",
          "sid": "194",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain sent by sales house to exchange — for publisher network O&O inventory (complete):**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "sales-house.com",
          "sid": "2",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain sent by sales house to exchange — for publisher network non-O&O inventory (incomplete, upstream origin unknown):**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 0,
      "nodes": [
        {
          "asi": "sales-house.com",
          "sid": "2",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain for sales house O&O site** — no SupplyChain is sent by the sales house to the exchange for its O&O inventory. The exchange sets `complete=1` with only its own node, since there is no upstream SupplyChain and the site is known to be O&O:

```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "exchange.com",
          "sid": "194",
          "hp": 1
        }
      ]
    }
  }
}
```

**Serialized tag format** (for tag-based integrations):

| Inventory | Serialized schain string |
|---|---|
| Managed publisher (to exchange) | `1.0,1!sales-house.com,1,1` |
| Publisher network O&O (to exchange) | `1.0,1!sales-house.com,2,1` |
| Publisher network non-O&O (to exchange) | `1.0,0!sales-house.com,2,1` |

---

#### Multi-Integration Scenario (Wrapper + Exchange Bidding)

An exchange sells the same publisher via two different supply paths: direct header bidding (exchange pays publisher directly) and exchange bidding through a secondary exchange (exchange pays secondary exchange, which pays publisher). The exchange must maintain **separate seller IDs** for each path — one seller ID must not represent both paths.

**exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "180000",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "domain": "examplepublisher.com"
    },
    {
      "seller_id": "180000B",
      "name": "EB Exchange",
      "seller_type": "INTERMEDIARY",
      "domain": "eb-exchange.com",
      "comment": "Publisher via Exchange Bidding"
    }
  ]
}
```

**eb-exchange.com/sellers.json:**
```json
{
  "version": "1.0",
  "sellers": [
    {
      "seller_id": "pub-1234",
      "name": "Example Publisher",
      "seller_type": "PUBLISHER",
      "is_passthrough": 1,
      "domain": "examplepublisher.com"
    }
  ]
}
```

**SupplyChain — inventory from EB Exchange path:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "eb-exchange.com",
          "sid": "pub-1234",
          "hp": 1
        },
        {
          "asi": "exchange.com",
          "sid": "180000B",
          "hp": 1
        }
      ]
    }
  }
}
```

**SupplyChain — inventory from direct header bidding path:**
```json
{
  "source": {
    "schain": {
      "ver": "1.0",
      "complete": 1,
      "nodes": [
        {
          "asi": "exchange.com",
          "sid": "180000",
          "hp": 1
        }
      ]
    }
  }
}
```

---

#### Deal Providers on Direct Publisher Inventory

In this scenario, deals from a publisher direct path and an ad network path are combined within a single bid request to avoid traffic duplication and reduce latency. The inventory remains under the control of the publisher. The payment flow differs depending on which deal wins:

- **Publisher deal wins:** Buyer → Exchange → Publisher
- **Ad network deal wins:** Buyer → Exchange → Publisher + Deal Provider (split), or Buyer → Exchange → Publisher → Deal Provider

Because the payment from the buyer to the publisher does not pass through the deal provider in either case, **the deal provider does not need to be included in the SupplyChain.**
