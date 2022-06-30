# Otter Society Akıllı Kontratı

### Terminoloji

Caller: Kullanıcının kripto-para cüzdanı.

### Halka Açık Mint

- Kullanıcı’nın kripto-para cüzdanı bu koşulları sağladığı takdirde halka açık mint’a katılabilir:
1. Akıllı kontrat olamaz.
    
    ```solidity
    /* EOA->A->B->C->D */
        /* if tx.origin and msg.sender are same,
         * msg.sender can NOT be a contract.
         */
        modifier callerIsUser() {
            require(
                tx.origin == msg.sender,
                "Otter Society :: Cannot be called by a contract"
            );
            _;
        }
    ```
    
2. Kontrat sahibi veya yöneticisi tarafından durdurulmadığı sürece.
    
    ```solidity
    /**
         * @dev Prevents a contract from calling itself, directly or indirectly.
         * Calling a `nonReentrant` function from another `nonReentrant`
         * function is not supported. It is possible to prevent this from happening
         * by making the `nonReentrant` function external, and making it call a
         * `private` function that does the actual work.
         */
        modifier nonReentrant() {
            // On the first call to nonReentrant, _notEntered will be true
            require(_status != _ENTERED, "ReentrancyGuard: reentrant call");
    
            // Any calls to nonReentrant after this point will fail
            _status = _ENTERED;
    
            _;
    
            // By storing the original value once again, a refund is triggered (see
            // https://eips.ethereum.org/EIPS/eip-2200)
            _status = _NOT_ENTERED;
        }
    ```
    
- Aşağıdaki şartlar sağlandığı takdirde kullanıcı NFT’yi mintleyebilir.
1. Halka açık mint kontrat sahibi ya da yöneticisi tarafından başlatılmalıdır.
    
    ```solidity
    require(publicSale, "Otter Society :: Not Yet Active.");
    ```
    
2. O ana kadar mint edilen NFTlerin sayısı `MAX_SUPPLY` değerini geçemez.
    
    ```solidity
    require(
                (totalSupply() + _quantity) <= MAX_SUPPLY,
                "Otter Society :: Beyond Max Supply"
            );
    ```
    
3. Kullanıcı, cüzdanı için en fazla `MAX_PUBLIC_MINT` kadar NFT mint edebilir. 
    
    ```solidity
    require(
                (totalPublicMint[msg.sender] + _quantity) <= MAX_PUBLIC_MINT,
                "Otter Society :: Minted maximum amount."
            );
    ```
    
4. $AVAX amount sent by the end-user must be equal to or higher than `PUBLIC_SALE_PRICE`. End-users can mint their NFTs through Kalao and/or SnowTrace (blockchain explorer of Avalanche’s C-chain)
Note: For $N$ NFTs, you must send at least `PUBLIC_SALE_PRICE * $N$` amount of $AVAX.
    
    ```solidity
    require(
                msg.value >= (PUBLIC_SALE_PRICE * _quantity),
                "Otter Society :: Not enough AVAX. "
            );
    ```
    
5. Kullanıcı tarafından gönderilen $AVAX miktarı `PUBLIC_SALE_PRICE` ’a eşit ya da daha yüksek bir miktar olmak zorundadır. Kullanıcılar NFT’leri Kalao ve/veya SnowTrace aracılığıyla mintleyebilirler.
**Not**: $N$adet NFT için, en az `PUBLIC_SALE_PRICE * $N$` kadar $AVAX gönderilmelidir.

### Whitelist Mint

- Kullanıcı, whitelist mint sürecine erişebilmek için aşağıdaki şartları sağlamalıdır:
1. Kullanıcı, akıllı kontrat olmadığı sürece,
    
    ```solidity
    /* EOA->A->B->C->D */
        /* if tx.origin and msg.sender are same,
         * msg.sender can NOT be a contract.
         */
        modifier callerIsUser() {
            require(
                tx.origin == msg.sender,
                "Otter Society :: Cannot be called by a contract"
            );
            _;
        }
    ```
    
2. Kontrat sahibi veya yöneticisi tarafından durdurulmadığı sürece,
    
    ```solidity
    /// @notice As an end-user, when the pause is set to 'false'
        /// you are allowed to access whitelist mint and public mint.
        modifier notPaused() {
            require(!pause, "Otter Society :: Contract is paused.");
            _;
        }
    ```
    
3. Kullanıcı, whitelist’e kayıtlı olduğu sürece.
    
    ```solidity
    modifier isWhitelisted(address _address) {
            require(whitelistedAddresses[_address], "You need to be whitelisted");
            _;
        }
    ```
    
- Aşağıdaki şartlar sağlandığı takdirde kullanıcı NFT’yi mintleyebilir.
1. Whitelist mint kontrat sahibi ya da yöneticisi tarafından başlatılmalıdır.
    
    ```solidity
    require(
                whiteListSale,
                "Otter Society :: White-list minting is on pause"
            );
    ```
    
2. O ana kadar mint edilen NFTlerin sayısı `MAX_SUPPLY_WHITELIST` değerini geçemez.
    
    ```solidity
    require(
                (totalSupply() + _quantity) <= MAX_SUPPLY_WHITELIST,
                "Otter Society :: Cannot mint beyond max supply"
            );
    ```
    
3. Kullanıcı, cüzdanı için en fazla `MAX_WHITELIST_MINT` kadar NFT mint edebilir. 
**Not**: Whitelist sahibi olan kullanıcılar cüzdanlarına `MAX_WHITELIST_MINT` ve `MAX_PUBLIC_MINT` toplamı kadar mint edebilirler.
    
    ```solidity
    require(
                (totalWhitelistMint[msg.sender] + _quantity) <= MAX_WHITELIST_MINT,
                "Otter Society :: Cannot mint beyond whitelist max mint!"
            );
    ```
    
4. Kullanıcı tarafından gönderilen $AVAX miktarı `WHITELIST_SALE_PRICE` ’a eşit ya da daha yüksek bir miktar olmak zorundadır. Kullanıcılar NFT’leri Kalao ve/veya SnowTrace aracılığıyla mintleyebilirler.
**Not**: $N$adet NFT için, en az `WHITELIST_SALE_PRICE * $N$` kadar $AVAX gönderilmelidir.
    
    ```solidity
    require(
                msg.value >= (WHITELIST_SALE_PRICE * _quantity),
                "Otter Society :: Payment is below the price"
            );
    ```
    

### Yönetici Mint

- Kullanıcı sadece kontrat yöneticisi ya da sahibi olabilir, Otter Society Yazılım Ekibi.
1. Kontrat sahibi ya da yöneticisi `TEAM_MINT_AMOUNT` kadar NFT adedini kendi cüzdanlarına mint edecek.
    
    ```solidity
    require(!teamMinted, "Otter Society :: Team already minted.");
    ```
    

### Withdraw

- Kullanıcı sadece kontrat yöneticis ya da sahibi olabilir.
1. Her çekme işleminde, %75 Otter Society ekibine gider, bu ücret road-map’i ve ekip maaşlarını karşılamak için kullanılır.
    
    ```solidity
    _withdraw(owner(), (fullBalance * 75) / 100);
    ```
    
2. Her çekme işleminde, %25 DAO cüzdanına gider, Bu cüzdan içerisindeki $AVAX Otter Society Yazılım Ekibi tarafından kullanılmayacaktır, Cüzdan içerisindeki her türlü bakiyenin kullanımına DAO karar verecektir.
    
    ```solidity
    _withdraw(marketDAOWallet, (fullBalance * 25) / 100);
    ```
    

### Telif Hakkı Payı

> Tüm NFT’ler satılmadan önce telif geliri: %10
Tüm NFT’ler satıldıktan sonra telif geliri: %5
**Not:**   Otter Society Yazılım Ekibi, ya da herhangi bir 3. parti’nin, telif hakkı gelirin kontratın herhangi bir hal ve/veya durumunda %20 üstüne çıkarması mümkün değildir.
> 

```solidity
	/// Reveal Otter Society on sold-out.
        /// Reduce royalty fee to 5%
        if (totalSupply() == MAX_SUPPLY) {
            isRevealed = true;
            _setDefaultRoyalty(marketDAOWallet, uint96(royaltyDividend / 2));
        }
```

*This documentation has been made & prepared by Otter Society Developer Team for public access and educational purposes. 

This documentation accurately represents The Otter Society NFT Smart Contract that will be used to host The Otter Society NFT Collection.*
