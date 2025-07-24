# CW-2981 Configuration Contract

An NFT contract that extends the EIP-2981 royalty standard with custom, obligatory configuration metadata for each token.

This contract builds on the pattern from cw2981-royalties (cw721-metadata-onchain). It implements all standard CW-721 and EIP-2981 royalty logic, but adds a requirement to include specific configuration details when minting a new token.

Royalty Queries
This contract exposes two standard query messages for royalty information, compliant with the CW-2981 specification.

```rust
#[cw_serde]
#[derive(QueryResponses)]
pub enum Cw2981QueryMsg {
    /// Called by a marketplace on sale to determine royalty payment.
    /// See https://eips.ethereum.org/EIPS/eip-2981
    #[returns(RoyaltiesInfoResponse)]
    RoyaltyInfo {
        token_id: String,
        // The sale price of the token.
        sale_price: Uint128,
    },
    /// Called by a marketplace to confirm the contract implements CW-2981.
    #[returns(CheckRoyaltiesResponse)]
    CheckRoyalties {},
}
The responses for these queries are:
```

```rust
#[cw_serde]
pub struct RoyaltiesInfoResponse {
    // The address that should receive the royalty payment.
    pub address: String,
    // The amount to be paid, calculated from the sale_price.
    pub royalty_amount: Uint128,
}

#[cw_serde]
pub struct CheckRoyaltiesResponse {
    // A boolean indicating that royalty payments should be checked.
    pub royalty_payments: bool,
}
Setting Metadata on Mint
To set royalty and configuration information, new fields are available in the extension parameter during the Mint execution message.

The Metadata struct includes optional royalty information and obligatory configuration fields.
```

```rust
#[cw_serde]
pub struct Metadata {
    // --- Optional Royalty Fields ---
    
    /// The percentage of the sale price to pay as a royalty (e.g., 10 for 10%).
    pub royalty_percentage: Option<u64>,
    
    /// The address to send royalty payments to. Can be a wallet, multisig, or another contract.
    pub royalty_payment_address: Option<String>,

    // --- Obligatory Configuration Fields ---

    /// The URL of the configuration's primary image.
    pub configuration_image_url: String,

    /// The SHA hash of the configuration image for verification.
    pub image_sha: String,
    
    /// The version of the Crossplane configuration.
    pub crossplane_version: String,

    // ... other standard optional metadata fields like name, description, etc.
}
```

# A Note on CheckRoyalties

For this contract, CheckRoyalties will always return royalty_payments: true. Because royalties are defined at the token level, any compliant marketplace should query RoyaltyInfo for every sale to see if a specific token requires a royalty payment. Contracts extending this one could implement more complex logic if, for example, they maintain a secondary index of which tokens have royalties. In this implementation, that is not necessary.
