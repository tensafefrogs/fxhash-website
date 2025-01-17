---
title: 'Integration guide'
date: '2021-12-31'
description: 'An integration guide of different fxhash components for the developers'
---



# Table of Contents


# Scope of the guide

This guide provides informations on how to integrate some of the components of fxhash into any external application. First of all, this guide currently targets the beta version, and so some informations might change as some components are updated. Because we want to encourage the development of ecosystems around applications in the decentralized world, we think applications should provide resources to developers in order to facilitate those integrations. This guide will point you to tools and cover best practices. This guide will always describe the integration for the latest version, but as it will get updated you will find thoses updates at the end of the article, if needed.


# Using the open GraphQL API

fxhash exposes a public GraphQL API (*the API will be open sourced in the upcoming days*). The API is exposed under the following URL: [https://api.fxhash.xyz/graphql](https://api.fxhash.xyz/graphql). You can interact with the API by sending HTTP POST requests complying to the GraphQL specifications. This is a very easy and flexible way to get only the data you need. This article briefly explains how to interact with the GraphQL APIs using HTTP requests: [https://graphql.org/graphql-js/graphql-clients/](https://graphql.org/graphql-js/graphql-clients/).

## Tooling

Some GraphQL clients facilitate the implementation of GraphQL requests:
* [graphql-request](https://www.npmjs.com/package/graphql-request) [JS]: minimal GraphQL client supporting Node and browsers for scripts or simple apps
* [Apollo Client](https://www.apollographql.com/docs/react/) [JS]: used by the fxhand frontend, state management, cache... some robust and versatile components fore more complexe applications
* [graphql-python/gql](https://github.com/graphql-python/gql) [Python]: simple library

The API enables [introspection](https://graphql.org/learn/introspection/), which means that you can use any tool to inspect the schema and explore a way to get the resources you need for your use-case:
* [Apollo Studio sandbox explorer](https://studio.apollographql.com/sandbox?endpoint=https%3A%2F%2Fapi.fxhash.xyz%2Fgraphql): UX friendly features to explore the schema **very easily** (recommended)
* [fxhash Graphiql endpoint](https://api.fxhash.xyz/graphiql): basic interface to test queries, less features than Apollo Sandbox Explorer

## Best practices

Try to compose your queries to only request the data you need. Each extra byte might not seem to have a huge impact individually, but when operating at a large scale it can quickly become a huge overload for the application and worsen the experience of all the users. If we all make the efforts to optimize the queries, we can reduce this load and improve the experience for all the community.

Also, try to think in terms of **query cost**. For instance, if your application needs to run lots of queries, and if those queries seem to be slow (+800ms), it's quite possible that they could be optimised. Check if other endpoints can provide the same informations using pagination for instance. Ultimately, feel free to reach out on [discord](https://discord.gg/wzqxfdCKCC) (channel `#3rd-party-integration`) if you encounter any issue / want to report a problem.


# Integrating fxhash resources

The Generative Tokens and the Gentks have some self-explanatory properties, however others may require more details to be used properly. This is particularly the case of the metadata of those tokens.

## Versioning

First of all, the metadata is versioned so that whenever a change is made to their definition, it reflects in their `version` field. Prior to version `0.2`, there wasn't any version field so if the `version` field is not defined in some metadata, one can assume that the version is `0.1`. This function ensures that given some metadata (let it be a Generative Token or a Gentk), it outputs its version:

```js
function getVersion(metadata) {
  return metadata.version ?? "0.1"
}
```

## Generative Tokens

These are the definitions for the metadata of a Generative for version 0.2:

```json
{
   "name": "Name of the Geberative Token",
   "description": "Description of the Generative Token",
   "childrenDescription": "Description of the Gentks generated",
   "tags": [ "list", "of", "tags" ],
   // URI to display the live version
   "artifactUri":"ipfs://QmWKdzyjJvZV2TFT6A9TTkBosGxvkmt7eTWcW338MpTgS9?fxhash=oo7KXsC8LbggF4wafcscSyiPGgCFbZZdVtGuBSv477bpjw76UpF",
   // URI to display the HQ preview
   "displayUri":"ipfs://QmcCbKcwLoPrfwPYzdbX1MEVgc91a5XdFb2vqNcu69Xt1T",
   // URI to display the thumbnail, a LQ version of the preview
   "thumbnailUri":"ipfs://QmWC7E3Sp8QzHN5EA33krcpuZv6Fi2CzjUQqZWdic2CB7E",
   // URI of the generator
   "generativeUri":"ipfs://QmWKdzyjJvZV2TFT6A9TTkBosGxvkmt7eTWcW338MpTgS9",
   // hash generated by fxhash backend to assert authenticity of the token
   "authenticityHash":"56972db4c0841a854c6a307f6fbfe5c699e6524ff28d3d82c5d46469a6efbaea",
   // hash used for the preview of the token
   "previewHash":"oo7KXsC8LbggF4wafcscSyiPGgCFbZZdVtGuBSv477bpjw76UpF",
   // capture settings
   "capture":{
      "mode":"VIEWPORT",
      "triggerMode":"FN_TRIGGER",
      "resolution":{
         "x":1080,
         "y":1080
      }
   },
   // other general-purpose settings
   "settings":{
      "exploration":{
         "preMint":{
            "enabled":true,
            "hashConstraints":null
         },
         "postMint":{
            "enabled":false,
            "hashConstraints":null
         }
      }
   },
   "symbol":"FXGEN",
   "decimals":0,
   "version":"0.2"
}
```

We will not describe the differences between prior versions and this one, but rather propose a generic-purpose function to get a display URL of the live version of a Generative Token regardless of its version:

```js
// get live display URL of a Generative Token
export function generativeLiveDisplayUrl(uri) {
  // you can use another gateway if you prefer
  const gateway = "https://gateway.fxhash.xyz/ipfs/"
  return gateway + metdata.artifactUri.substring(7)
}
```

This function removes the `ipfs://` at the beginning of the URI and composes a display URL based on the result.

## Gentks

Gentks follow approximately the same construction:

```json
{
  "name": "Name of the Gentk",
  // transaction hash used to generate the Gentk
  "iterationHash": "opXnGtQiUMfyKL2AHq6c13E3tg7fxUKx1eTD4UoxFdVWBR1YuE8",
  "description": "Description (same as the .childrenDescription field of its parent)",
  "tags": [ "list", "of", "tags" ], // same as its parent
  // URI to the generator, cannot be displayed alone
  "generatorUri": "ipfs://QmWKdzyjJvZV2TFT6A9TTkBosGxvkmt7eTWcW338MpTgS9",
  // URI to display the live version
  "artifactUri": "ipfs://QmWKdzyjJvZV2TFT6A9TTkBosGxvkmt7eTWcW338MpTgS9?fxhash=opXnGtQiUMfyKL2AHq6c13E3tg7fxUKx1eTD4UoxFdVWBR1YuE8",
  // HQ preview
  "displayUri": "ipfs://QmZ1SD15ynTmCPCGbW6tgCrTM2go7J1jFcLJcv4sSPgtSG",
  // LQ preview
  "thumbnailUri": "ipfs://QmU2oJGJKq5jQsKKxBf2bVhSnWoYS28owfzyXx1Mb34zyF",
  // hash generated by fxhash backend to assert authenticity of the token
  "authenticityHash": "aa04e03a6fee5ebeb9ab4b3c38acd32b0a94f1abe1d47784e50bccc9152c6f56",
  // list of attributes for the token
  "attributes": [
    {
      "name": "Attribute 1",
      "value": "it's a string"
    },
    {
      "name": "Attribute 2",
      "value": 17
    }
  ],
  "decimals": 0,
  "symbol": "GENTK",
  "version": "0.2"
}
```

The exact same function can be used to display the live version of a Gentk:

```js
// get live display URL of a Gentk
export function gentkLiveDisplayUrl(uri) {
  // you can use another gateway if you prefer
  const gateway = "https://gateway.fxhash.xyz/ipfs/"
  return gateway + metdata.artifactUri.substring(7)
}
```

## Best practices

* The thumbnails are 300x300 png images. They should be used instead of the HQ preview if possible, to reduce the cost of the gateway.


# Inspecting the contracts directly

If the solutions provided above don't meet your needs, you can use other tools to inspect the contract's data with as much control as you need (for instance general-purpose public APIs such as [tzkt](https://api.tzkt.io/)).

These are the addresses of the smart contracts used by fxhash

* `issuer` (Generative Tokens): KT1XCoGnfupWk7Sp8536EfrxcP73LmT68Nyr
* `gentks` (FA2 implementation): KT1KEa8z6vWXDJrVqtMrAeDVzsvxat3kHaCE
* `marketplace`: KT1Xo5B7PNBAeynZPmca4bRh6LQow4og1Zb9
* `user register`: KT1Ezht4PDKZri7aVppVGT4Jkw39sesaFnww
* `token moderation`: KT1HgVuzNWVvnX16fahbV2LrnpwifYKoFMRd
* `user moderation`: KT1TWWQ6FtLoosVfZgTKV2q68TMZaENhGm54
* `cycles`: KT1ELEyZuzGXYafD2Gar6iegZN1YdQR3n3f5


# About the moderation contracts

You can get some details on the moderation systems:

* [User Moderation system](https://github.com/fxhash/system-descriptions/blob/master/USER_MODERATION.md)
* [Generative Token moderation system](https://www.fxhash.xyz/articles/moderation-system)


# Updates

## 05/01/2022

* A new issuer contract was deployed and will replace the new one. Some storage changes were made to optimize the space. Metadata was moved into the ledger. Ledger is now referenced by the issuer ID only, and the creator is now a field. This simplifies lookups on the contract side.
  - issuer: `KT1AEVuykWeuuFX7QkEAMNtffzwhe1Z98hJS` -> `KT1XCoGnfupWk7Sp8536EfrxcP73LmT68Nyr`
  - user moderation: `KT1CgsLyNpqFtNw3wdfGasQYZYfgsWSMJfGo` -> `KT1TWWQ6FtLoosVfZgTKV2q68TMZaENhGm54`
  - user moderation: `KT1BQqRn7u4p1Z1nkRsGEGp8Pc92ftVFqNMg` -> `KT1HgVuzNWVvnX16fahbV2LrnpwifYKoFMRd`