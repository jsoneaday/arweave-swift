# Arweave Client SDK for Swift

![](https://img.shields.io/badge/Swift-5.5-orange.svg)
[![GitHub license](https://img.shields.io/badge/license-MIT-lightgrey.svg)](https://github.com/lukereichold/arweave-swift/blob/master/LICENSE) 
[![SPM compatible](https://img.shields.io/badge/spm-compatible-brightgreen.svg?style=flat)](https://swift.org/package-manager)
[![Twitter](https://img.shields.io/badge/twitter-@lreichold-blue.svg?style=flat)](https://twitter.com/lreichold)

A lightweight Swift client for the Arweave blockchain, providing type safety for interacting with the Arweave API

⚠️ _This `master` branch requires requires Xcode 13 / Swift 5.5. For supporting older versions, see the [stable branch](https://github.com/lukereichold/arweave-swift/tree/stable)._

## Installation

To install via [Swift Package Manager](https://swift.org/package-manager), add `Arweave` to your `Package.swift` file. Alternatively, [add it to your Xcode project directly](https://developer.apple.com/documentation/xcode/adding_package_dependencies_to_your_app).

```swift
let package = Package(
    ...
    dependencies: [
        .package(url: "https://github.com/lukereichold/arweave-swift.git", from: "1.1.0")
    ],
    ...
)
```

Then import `Arweave` wherever you’d like to use it:

```swift
import Arweave
```

## Demo

See the included demo app, written in [SwiftUI](https://developer.apple.com/xcode/swiftui/), which dynamically creates a `Wallet` object from an existing Arweave JWK keyfile and uses an [iOS share extension](https://developer.apple.com/design/human-interface-guidelines/ios/extensions/sharing-and-actions/) to create and submit a new data transaction containing the data of a given page in Safari.

![](demo.gif)


## Usage

### Specify Custom Arweave Node (optional)

```swift
Arweave.baseUrl = URL(string: "https://myArweaveNode.net:443")!
```

Craft a custom `URL` to specify host, port, and protocol of target node. All subsequent requests will automatically be made to this custom host.

### Wallets and Keys

#### Creating a Wallet from an existing JWK keyfile

```swift
guard let keyFileData = try? Data(contentsOf: keyFileUrl) else { return }

let wallet = Wallet(jwkFileData: keyFileData)
```

#### Get a wallet's public address

```swift
wallet.address
/// 1seRanklLU_1VTGkEk7P0xAwMJfA7owA1JHW5KyZKlY
```

#### Check wallet balance (async)

All wallet balances are returned using [winston](https://docs.arweave.org/developers/server/http-api#ar-and-winston) units. 
```swift
let balance = try await wallet.balance()
```

#### Convert amounts between AR and winston units
```swift
var transferAmount = Amount(value: 1, unit: .AR)
let amtInWinston = transferAmount.converted(to: .winston)
XCTAssertEqual(amtInWinston.value, 1000000000000, accuracy: 0e-12) // ✅

transferAmount = Amount(value: 2, unit: .winston)
let amtInAR = transferAmount.converted(to: .AR)
XCTAssertEqual(amtInAR.value, 0.000000000002, accuracy: 0e-12) // ✅
```

#### Fetch the last transaction ID for a given wallet (async)

```swift
let lastTxId = try await wallet.lastTransactionId()
```

### Transactions

Transactions are the building blocks of the Arweave permaweb. They can send [AR](https://docs.arweave.org/developers/server/http-api#ar-and-winston) between wallet addresses or store data on the Arweave network.

#### Create a Data Transaction

Data transactions are used to store data on the Arweave permaweb and can contain any arbitrary data.

```swift
let data = "<h1>Hello World!</h1>".data(using: .utf8)!
let transaction = Transaction(data: data)
```

#### Create a wallet-to-wallet Transaction

```swift
let targetAddress = Address(address: "someOtherWalletAddress")
let transferAmount = Amount(value: 500, unit: .winston)

let transaction = Transaction(amount: transferAmount, target: targetAddress)
```

#### Modifying an existing Transaction

Metadata can be optionally added to transactions through tags, these are simple key/value attributes that can be used to document the contents of a transaction or provide related data.

```swift
let tag = Transaction.Tag(name: "myTag", value: "myValue")
transaction.tags.append(tag)
```

#### Signing and Submitting a Transaction

The data and wallet-to-wallet transaction initializers above simply create an unsigned `Transaction` object. To be submitted to the network, however, each `Transaction` must first be signed.

```swift

let transaction = Transaction(data: data)
let signedTx = try await transaction.sign(with: wallet)
try await signed.commit()
```


⚠️ **Modifying a transaction object after signing it will invalidate the signature**, this will cause it to be rejected by the network if submitted in that state. Transaction prices are based on the size of the data field, so modifying the data field after a transaction has been created isn't recommended as you'll need to manually update the price.

The transaction ID is a hash of the transaction signature, so a transaction ID can't be known until its contents are finalized and it has been signed.

#### Get a Transaction status (async)

```swift
let txStatus = try await Transaction.status(of: exampleTxId)

/// Arweave.Transaction.Status.accepted(data: Arweave.Transaction.Status.Data(block_height: 502761, block_indep_hash: "V6pCKSyeQiqICWKM2G_zkQ8SCA_WKnZoVGOD8eKFV_xozoWS9xPFgncxnMWjtFao", number_of_confirmations: 8655))
```

#### Fetch Transaction content for a given ID (async)

```swift
let tx = try await Transaction.find(exampleTxId)
```

#### Fetch Transaction data (async)

We can get the transaction data (represented as a base64 URL encoded string) for a given transaction ID without having to fetch the entire Transaction object.

```swift
let txData = try await Transaction.data(for: exampleTxId)
```

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/lukereichold/arweave-swift/issues).
