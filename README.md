# @SYNC-Network/collection-lists (beta)

This is a fork of the Uniswap Token list standard designed to work with ERC721 and ERC1155 token standards and not to conflict with Uniswap's ERC20 token lists.

They are not compatible with each other by design to improve user experience and meet the specific needs of the NFT community.

If you are familiar with Uniswap Token Lists, the key differences are the contracts are under "collections" instead of "tokens" and fields for symbol and decimals have been removed to make the lists lighter, faster, and prevent confusion loading lists for dapps designed for ERC20's and vice verca.

[![Tests](https://github.com/SYNC-Network/collection-lists/workflows/Tests/badge.svg)](https://github.com/SYNC-Network/collection-lists/actions?query=workflow%3ATests)
[![npm](https://img.shields.io/npm/v/@uniswap/token-lists)](https://unpkg.com/@uniswap/token-lists@latest/)

This package includes a JSON schema for collection lists, and TypeScript utilities for working with collection lists.

The JSON schema represents the technical specification for a collection list which can be used in a dApp interface, such as the Neon Rain Interface.

## What are Collection lists?

Neon Rain collection lists are specifications for lists of NFT collection metadata (e.g. address, icon, etc...) that can be used by any dApp interfaces that need one or more NFT collections lists.

Anyone can create and maintain a Collection list, as long as they follow the specification.

Specifically, an instance of a collection list is a [JSON](https://www.json.org/json-en.html) blob that contains a list of
[ERC721](https://github.com/ethereum/eips/issues/721) and [ERC1155](https://github.com/ethereum/eips/issues/1155) token metadata for use in dApp user interfaces.
Token list JSON must validate against the [JSON schema](https://json-schema.org/) in order to be used in the Neon Rain Interface.
Collections on collection lists, and collection lists themselves, are tagged so that users can easily find NFT collections.

## JSON Schema $id

The JSON schema ID is [https://uniswap.org/tokenlist.schema.json](https://uniswap.org/tokenlist.schema.json)

## Validating collection lists

This package does not include code for collection list validation. You can easily do this by including a library such as
[ajv](https://ajv.js.org/) to perform the validation against the JSON schema. The schema is exported from the package
for ease of use.

```typescript

import { schema } from '@SYNC-Network/collection-lists'
import Ajv from 'ajv'
import addFormats from 'ajv-formats'
import fetch from 'node-fetch'

const ARBITRUM_LIST = 'https://bridge.arbitrum.io/token-list-42161.json'

async function validate() {
  const ajv = new Ajv({ allErrors: true, verbose: true })
  addFormats(ajv)
  const validator = ajv.compile(schema);
  const response = await fetch(ARBITRUM_LIST)
  const data = response.json()
  const valid = validator(data)
  if (valid) {
    return valid
  }
  if (validator.errors) {
    return validator.errors.map(error => {
      delete error.data
      return error
    })
  }
}

validate()
  .then(console.log)
  .catch(console.error)

```

## Authoring collection lists

### Manual

The best way to manually author collection lists is to use an editor that supports JSON schema validation. Most popular
code editors do, such as [IntelliJ](https://www.jetbrains.com/help/idea/json.html#ws_json_schema_add_custom) or
[VSCode](https://code.visualstudio.com/docs/languages/json#_json-schemas-and-settings). Other editors
can be found [here](https://json-schema.org/implementations.html#editors).

The schema is registered in the [SchemaStore](https://github.com/SchemaStore/schemastore), and any file that matches
the pattern `*.collectionlist.json` should
[automatically utilize](https://www.jetbrains.com/help/idea/json.html#ws_json_using_schemas)
the JSON schema for the [supported text editors](https://www.schemastore.org/json/#editors).

In order for your token list to be able to be used, it must pass all JSON schema validation.

### Automated

If you want to automate token listing, e.g. by pulling from a smart contract, or other sources, you can use this
npm package to take advantage of the JSON schema for validation and the TypeScript types.
Otherwise, you are simply working with JSON. All the usual tools apply, e.g.:

```typescript
import { TokenList, schema } from '@SYNC-Network/collection-lists'

// generate your token list however you like.
const myList: TokenList = generateMyTokenList();

// use a tool like `ajv` to validate your generated token list
validateMyTokenList(myList, schema);

// print the resulting JSON to stdout
process.stdout.write(JSON.stringify(myList));
```

## Semantic versioning

Lists include a `version` field, which follows [semantic versioning](https://semver.org/).

List versions must follow the rules:

- Increment major version when collections are removed
- Increment minor version when collections are added
- Increment patch version when collections already on the list have minor details changed (name, symbol, logo URL, etc)

Changing a collection address or chain ID is considered both a remove and an add, and should be a major version update.

Note that list versioning is used to improve the user experience, but not for security, i.e. list versions are not meant
to provide protection against malicious updates to a token list; i.e. the list semver is used as a lossy compression
of the diff of list updates. List updates may still be diffed in the client dApp.

## Deploying your list

Once you have authored the list, you can make it available at any URI. Prefer pinning your list to IPFS
(e.g. via [pinata.cloud](https://pinata.cloud)) and referencing the list by an ENS name that resolves to the
[contenthash](https://eips.ethereum.org/EIPS/eip-1577).

If hosted on HTTPS, make sure the endpoint is configured to send an access-control-allow-origin header to avoid CORS errors.

### Linking an ENS name to the list

An ENS name can be assigned to an IPFS hash via the [contenthash](https://eips.ethereum.org/EIPS/eip-1577) text record.
This is the preferred way of referencing your list.

## Examples

You can find a simple example of a token list in [test/schema/example.tokenlist.json](test/schema/example.tokenlist.json).

A snapshot of the Uniswap default list encoded as a token list is found in [test/schema/bigexample.tokenlist.json](test/schema/bigexample.tokenlist.json).
