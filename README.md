## Contract Resolver

This repo contains one contract and a few others to showcase how to make use of it.
The main one to pay attention to is [Resolver](./contracts/Resolver.cdc) which is a basic
contract interface to permit borrowing a contract and resolve views ontop of it.

With this new interface, we can do things like resolve an NFT contract's collection data
without having the NFT itself, and without causing something to panic because we try to access a
method that doesn't exist or import a contract blindly through some kind of script or transaction template

[ExampleNFT](./contracts/standard/ExampleNFT.cdc) contains an example implementation:
```cadence
pub contract ExampleNFT: NonFungibleToken, ResolverContract {
    ...

    pub fun resolveView(_ view: Type): AnyStruct? {
        switch view {
            case Type<MetadataViews.NFTCollectionData>():
                return MetadataViews.NFTCollectionData(
                    storagePath: ExampleNFT.CollectionStoragePath,
                    publicPath: ExampleNFT.CollectionPublicPath,
                    providerPath: /private/exampleNFTCollection,
                    publicCollection: Type<&ExampleNFT.Collection{ExampleNFT.ExampleNFTCollectionPublic}>(),
                    publicLinkedType: Type<&ExampleNFT.Collection{ExampleNFT.ExampleNFTCollectionPublic,NonFungibleToken.CollectionPublic,NonFungibleToken.Receiver,MetadataViews.ResolverCollection}>(),
                    providerLinkedType: Type<&ExampleNFT.Collection{ExampleNFT.ExampleNFTCollectionPublic,NonFungibleToken.CollectionPublic,NonFungibleToken.Provider,MetadataViews.ResolverCollection}>(),
                    createEmptyCollectionFunction: (fun (): @NonFungibleToken.Collection {
                        return <-ExampleNFT.createEmptyCollection()
                    })
                )
            ...
        }
        return nil
    }

    pub fun getViews(): [Type] {
        return [
            Type<MetadataViews.NFTCollectionData>(),
            ...
        ]
    }
```

And a sample script to go with it!
```
import Resolver from 0xf8d6e0586b0a20c7
import MetadataViews from 0xf8d6e0586b0a20c7

pub fun main(addr: Address, name: String): AnyStruct? {
  let t = Type<MetadataViews.NFTCollectionData>()
  let borrowedContract = getAccount(addr).contracts.borrow<&Resolver>(name: name) ?? panic("contract could not be borrowed")

  let view = borrowedContract.resolveView(t)
  if view == nil {
    return nil
  }

  let cd = view! as! MetadataViews.NFTCollectionData
  return cd.storagePath
}
```

```
{"domain":"storage","identifier":"exampleNFTCollection"}
```