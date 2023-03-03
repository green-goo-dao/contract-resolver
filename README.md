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

And a [sample script](https://runflow.pratikpatel.io?code=aW1wb3J0IFJlc29sdmVyIGZyb20gMHhmOGQ2ZTA1ODZiMGEyMGM3CmltcG9ydCBNZXRhZGF0YVZpZXdzIGZyb20gMHhmOGQ2ZTA1ODZiMGEyMGM3CgpwdWIgZnVuIG1haW4oYWRkcjogQWRkcmVzcywgbmFtZTogU3RyaW5nKTogQW55U3RydWN0PyB7CiAgbGV0IHQgPSBUeXBlPE1ldGFkYXRhVmlld3MuTkZUQ29sbGVjdGlvbkRhdGE%2BKCkKICBsZXQgYm9ycm93ZWRDb250cmFjdCA9IGdldEFjY291bnQoYWRkcikuY29udHJhY3RzLmJvcnJvdzwmUmVzb2x2ZXI%2BKG5hbWU6IG5hbWUpID8%2FIHBhbmljKCJjb250cmFjdCBjb3VsZCBub3QgYmUgYm9ycm93ZWQiKQoKICBsZXQgdmlldyA9IGJvcnJvd2VkQ29udHJhY3QucmVzb2x2ZVZpZXcodCkKICBpZiB2aWV3ID09IG5pbCB7CiAgICByZXR1cm4gbmlsCiAgfQoKICBsZXQgY2QgPSB2aWV3ISBhcyEgTWV0YWRhdGFWaWV3cy5ORlRDb2xsZWN0aW9uRGF0YQogIHJldHVybiBjZC5zdG9yYWdlUGF0aAp9&network=emulator&args=eyJhZGRyIjoiMHhmOGQ2ZTA1ODZiMGEyMGM3IiwibmFtZSI6IkV4YW1wbGVORlQifQ%3D%3D) to go with it!
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