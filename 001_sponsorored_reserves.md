# Migrating Mesh accounts for releasing locked funds post Stellar Protocol 15 adoption

## An overview and some background context

[Mesh](app.mesh.trade) makes use of the Stellar blockchain for facilitating platform trading, transfers and other financial transactions.

When creating an account on Stellar a minimum of 1 [XLM](https://www.stellar.org/lumens) has to be funded to make the account active on the network, this contributes to the Stellar [minimum required balance](https://developers.stellar.org/docs/glossary/minimum-balance/). Any additional account entry, such as [Trustlines](https://developers.stellar.org/docs/issuing-assets/anatomy-of-an-asset/#trustlines) and [Signers](https://developers.stellar.org/docs/glossary/accounts/#signers), increase the required reserves by 0.5 XLM.

Before the introduction of [Protocol 15](https://www.stellar.org/developers-blog/protocol-15-upgrade-complete) there was no native way to sponsor these reserves. In order to provide a seamless experience for interacting with the Stellar chain, Mesh sponsored all account reserve requirements by making a small transfer into your account every time an increase in reserves was required. This balance was then locked by the network but still contributed to your account balance - more on this later.

Since the implementation of [sponsored reserves](https://developers.stellar.org/docs/glossary/sponsored-reserves/), Mesh uses the sponsored reserves mechanism for funding your minimum reserve balance requirements.
This eliminates the need for a "transfer and lock" mechanism for any and all entries added to your account after 01 March 2022.

## What is the effect of the migration 1

Mesh will be taking over the sponsorship requirements for all our user accounts with any locked XLM balance on 12 August 2022.

If your account has a balance of XLM that you can't currently transfer or sell, this balance will become available to you after the migration. Mesh will effectively donate this previously locked value to our users.

No user action is required.

## How does the migration work

The implementation is based on the official Stellar documentation for [transferring sponsorships](https://developers.stellar.org/docs/glossary/sponsored-reserves/#transferring-sponsorship). A high level summary of what will happen during your account migration is described below:

The migration has to revoke the "self" sponsorship of the required minimum reserves and transfer the obligation of sponsoring the minimum reserves to the Mesh account.

This is achieved by "sandwiching" a set of [Revoke sponsorship](https://developers.stellar.org/docs/start/list-of-operations/#revoke-sponsorship) operations between a [Begin sponsoring future reserves](https://developers.stellar.org/docs/start/list-of-operations/#begin-sponsoring-future-reserves) operation and an [End sponsoring future reserves](https://developers.stellar.org/docs/start/list-of-operations/#end-sponsoring-future-reserves) operation.

The operations in the transaction will look something like this

```golang
    []txnbuild.Operation{
        // start the sandwich
        &txnbuild.BeginSponsoringFutureReserves{
            SourceAccount: MESH_ACCOUNT_ADDRESS,
            SponsoredID:   YOUR_ACCOUNT_ADDRESS,
        },

        // transfer account opening reserve requirement
        &txnbuild.RevokeSponsorship{
            SourceAccount:   YOUR_ACCOUNT_ADDRESS,
            SponsorshipType: txnbuild.RevokeSponsorshipTypeAccount,
            Account:         YOUR_ACCOUNT_ADDRESS,
        },

        // transfer signatory reserve requirements
        &txnbuild.RevokeSponsorship{
            SourceAccount:   YOUR_ACCOUNT_ADDRESS,
            SponsorshipType: txnbuild.RevokeSponsorshipTypeSigner,
            Signer: ...,
        }

        // transfer trust line reserve requirements
        &txnbuild.RevokeSponsorship{
            SourceAccount:   YOUR_ACCOUNT_ADDRESS,
            SponsorshipType: txnbuild.RevokeSponsorshipTypeTrustLine,
            TrustLine: ...,
        }

        // end the sandwich
        &txnbuild.EndSponsoringFutureReserves{
            SourceAccount: cleanUpTargetAcc.LedgerID.String(),
        },
    }
```

The transaction is then signed by the Mesh account and the default signatory on your account and submitted to the network.

## Who is effected

All Mesh users who registered their accounts before 01 March 2022. All effected users will be notified.
