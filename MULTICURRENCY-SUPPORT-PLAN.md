# Demux-Server Multi-Cryptocurrency Support Plan

This document outlines the comprehensive plan for modifying the Demux-Server fork to enable support for multiple cryptocurrencies in addition to the existing MobileCoin (MOB) implementation.

## Current State

Demux's payment system currently only supports MobileCoin (MOB) as its cryptocurrency for payments between users. The implementation is tightly coupled to MobileCoin-specific components.

## Target State

The goal is to create an abstraction layer that allows Demux-Server to support multiple cryptocurrencies, starting with adding Nano (XNO) alongside MobileCoin.

## Files and Components Requiring Modification

### 1. Profile Data Definition
**File:** `service/src/main/proto/org/signal/chat/profile.proto`
- The `payment_address` field currently only supports MobileCoin wallet ID
- Need to modify to support multiple cryptocurrency wallet addresses

### 2. Currency Configuration
**File:** `service/config/sample.yml`
- Current configuration:
```yaml
paymentsService:
  userAuthenticationTokenSharedSecret: secret://paymentsService.userAuthenticationTokenSharedSecret
  paymentCurrencies:
    # list of symbols for supported currencies
    - MOB
  externalClients:
    fixerApiKey: secret://paymentsService.fixerApiKey
    coinGeckoApiKey: secret://paymentsService.coinGeckoApiKey
    coinGeckoCurrencyIds:
      MOB: mobilecoin
```

### 3. Currency Conversion Logic
**File:** `service/src/main/java/org/whispersystems/textsecuregcm/currency/CurrencyConversionManager.java`
- Handles currency conversion via CoinGecko API
- Will need modification to handle additional cryptocurrency price lookups

### 4. CoinGecko Client
**File:** `service/src/main/java/org/whispersystems/textsecuregcm/currency/CoinGeckoClient.java`
- Retrieves cryptocurrency pricing from CoinGecko
- Already designed to work with different currencies via the `currencyIdsBySymbol` map

### 5. Authentication Token for Payments
**File:** `service/config/sample-secrets-bundle.yml`
- Contains MobileCoin specific authentication:
```yaml
paymentsService.userAuthenticationTokenSharedSecret: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA= # base64-encoded 32-byte secret shared with MobileCoin services used to generate auth tokens for Demux users
```

### 6. Payments GRPC Service
**File:** `service/src/main/java/org/whispersystems/textsecuregcm/grpc/PaymentsGrpcService.java`
- Handles payment-related gRPC requests
- Would need to be extended to handle different cryptocurrency types

### 7. Additional Files and Components
- **Transaction Processing Logic**: Any code handling transaction verification
- **Receipt & Notification Services**: Systems that notify users about transactions
- **Database Schemas**: Tables that store payment-related information
- **Backup and Restore Systems**: To handle multiple wallet types
- **Privacy Components**: Ensuring privacy features work with all cryptocurrencies

## Implementation Plan

### 1. Create an Abstraction Layer for Cryptocurrencies
- Define a `CryptocurrencyType` enum or class to replace hardcoded references to MobileCoin/MOB
- Implement cryptocurrency-specific wallet handling for each supported type
- Create interfaces for common operations (e.g., `validateAddress`, `formatAmount`, `createWallet`)
- Implement cryptocurrency-specific providers that implement these interfaces
- Design factory pattern for creating appropriate handlers based on cryptocurrency type

### 2. Database Schema Changes
- Update database schemas to support multiple cryptocurrency wallets per user
- Add cryptocurrency type identifiers to transaction records
- Ensure proper indexing for efficient queries across cryptocurrency types
- Consider migration plan for existing users with MobileCoin wallets

### 3. Modify Profile Schema
- Change the `payment_address` field in the profile proto to handle multiple wallet types
- Consider options:
  - Use a repeated field of wallet entries with type identifiers
  - Create separate fields for each supported cryptocurrency
  - Use a serialized structure with cryptocurrency type mapping
- Update serialization/deserialization logic
- Add validation for each cryptocurrency address format

### 4. Update Configuration System
- Extend the `paymentsService` configuration to include Nano XNO:
```yaml
paymentsService:
  paymentCurrencies:
    - MOB
    - XNO
  externalClients:
    coinGeckoCurrencyIds:
      MOB: mobilecoin
      XNO: nano
  currencyFeatures:
    MOB:
      requiresAuthentication: true
      minimumTransactionAmount: 0.0001
      maximumTransactionAmount: 100
    XNO:
      requiresAuthentication: false
      minimumTransactionAmount: 0.00001
      maximumTransactionAmount: 1000
  addressValidationPatterns:
    XNO: "^(nano|xrb)_[13][13-9a-km-uw-z]{59}$"
```
- Add feature flags for enabling/disabling specific cryptocurrencies
- Create configuration for transaction limits and fees per cryptocurrency
- Add dynamic configuration options for updates without server restart

### 5. Authentication and Security Updates
- Implement Nano-specific authentication if needed alongside MobileCoin authentication
- Create cryptocurrency-specific security validation
- Update rate limiting to consider cryptocurrency types
- Implement fraud detection patterns suitable for each cryptocurrency
- Update key management systems to handle multiple cryptocurrency wallets

### 6. Update Currency Conversion Logic
- Ensure CurrencyConversionManager can handle and convert XNO prices
- Make sure CoinGeckoClient properly fetches XNO pricing data
- Add fallback pricing sources for each cryptocurrency
- Implement caching strategies optimized for each cryptocurrency's volatility
- Add alerting for significant price movements

### 7. Transaction Processing Updates
- Create handlers for initiating transactions in different cryptocurrencies
- Implement verification logic specific to each blockchain
- Add monitoring for transaction status across multiple networks
- Implement fee estimation for each cryptocurrency
- Create retry mechanisms for failed transactions

### 8. Front-End Application Changes
- **API Changes**: 
  - Modify payment-related API endpoints to support cryptocurrency type specification
  - Add endpoints for retrieving supported cryptocurrency types
  - Update wallet address validation logic for different cryptocurrencies
  - Create new endpoints for cryptocurrency-specific operations

- **UI/UX Components**:
  - Add cryptocurrency selection options in payment setup screens
  - Create UX for managing multiple cryptocurrency wallets
  - Update transaction history to clearly indicate cryptocurrency types
  - Implement currency-specific QR code generation and scanning
  - Design UX for displaying exchange rates between cryptocurrencies
  - Add educational elements about each supported cryptocurrency

- **Mobile Client Updates**:
  - Android and iOS apps will need to be updated to support multiple wallet types
  - Implement cryptocurrency-specific functionality (address validation, display formatting)
  - Update local storage to handle multiple wallet configurations
  - Implement secure key storage for each cryptocurrency type
  - Add offline transaction preparation where applicable

- **Desktop Client Updates**:
  - Similar updates to mobile clients for the desktop applications
  - Optimize cryptocurrency management UI for larger screens

- **Client-Server Protocol**:
  - Update protocol to include cryptocurrency type identifiers in payment-related messages
  - Ensure backward compatibility for clients that only support MobileCoin
  - Version API endpoints to manage transitions
  - Add capability negotiation to determine client cryptocurrency support

### 9. Documentation and Support
- Update API documentation with multi-cryptocurrency support details
- Create user-facing documentation explaining cryptocurrency options
- Provide guidelines for wallet security for each cryptocurrency
- Document compatibility matrices for clients and server versions
- Create troubleshooting guides for transaction issues

### 10. Analytics and Monitoring
- Update monitoring systems to track usage of different cryptocurrencies
- Add analytics for cryptocurrency selection patterns
- Monitor transaction success rates per cryptocurrency
- Create alerts for unusual transaction patterns
- Track conversion rates between supported cryptocurrencies

## Implementation Strategy

The implementation should be done in a way that maintains backward compatibility with existing MobileCoin functionality while adding support for Nano XNO and providing a framework for adding other cryptocurrencies in the future.

### Back-End Strategy
- Create a pluggable architecture for cryptocurrency support
- Implement feature flags to control enabling/disabling specific cryptocurrencies
- Phase rollout starting with server-side changes followed by front-end updates
- Use feature toggles to enable different cryptocurrencies for specific user groups
- Consider canary deployments for testing with limited user base

### Front-End Strategy
- Use progressive enhancement approach to add new cryptocurrency options
- Implement feature detection to avoid breaking older clients
- Create fallback behavior for legacy clients that don't recognize the new cryptocurrency types
- Design UI to gracefully handle addition of new cryptocurrencies in the future
- Implement client-side feature flags synchronized with server configuration

### Data Migration Strategy
- Plan for migrating existing MobileCoin users to the new multi-wallet schema
- Create scripts for database schema updates
- Implement data backfills for new cryptocurrency fields
- Test migration on staging environments before production

## Security Considerations

### Wallet Security
- Ensure proper encryption for all wallet data
- Implement different security models based on cryptocurrency requirements
- Consider cold storage options for high-value cryptocurrencies
- Create secure backup mechanisms for all wallet types

### Transaction Security
- Implement proper validation for all transaction types
- Create monitoring for unusual transaction patterns
- Ensure privacy guarantees are maintained across cryptocurrencies
- Implement rate limiting specific to each cryptocurrency

### Compliance and Regulatory Considerations
- Review regulatory requirements for each supported cryptocurrency
- Implement appropriate KYC/AML procedures if required
- Consider geographic restrictions for certain cryptocurrencies
- Document compliance approach for each supported currency

## Performance Considerations
- Benchmark transaction processing for each cryptocurrency
- Optimize database queries for multi-cryptocurrency support
- Consider caching strategies for cryptocurrency conversion rates
- Ensure scalability for increasing number of supported cryptocurrencies
- Test performance under high transaction volumes across multiple currencies

## Testing Considerations

- Unit tests for currency conversion with multiple currencies
- Integration tests for wallet operations with different cryptocurrencies
- End-to-end tests simulating payments between users with different cryptocurrencies
- Compatibility tests with older client versions
- Cross-platform testing on all supported client platforms
- UI/UX testing for cryptocurrency selection and management workflows
- Security testing for each cryptocurrency implementation
- Performance testing under various transaction loads
- Compliance testing for regulatory requirements
- Disaster recovery testing including backup/restore of multiple wallet types
- Chaos testing to ensure resilience during network/service failures

## Third-Party Integration Considerations
- Update integrations with payment processors if applicable
- Coordinate with cryptocurrency wallet providers for proper implementations
- Consider exchange integrations for currency conversion
- Establish relationships with block explorers for transaction verification
- Plan for blockchain node hosting or third-party API usage for each cryptocurrency

## Rollout and Monitoring Plan
- Define metrics for successful implementation
- Create phased rollout schedule
- Implement enhanced monitoring during initial deployment
- Establish feedback channels for users
- Define rollback procedures if issues are encountered
- Plan for progressive enablement of cryptocurrencies

## Future Extensibility
- Design the system to easily accommodate additional cryptocurrencies
- Document process for adding new cryptocurrency support
- Create templates for cryptocurrency-specific components
- Implement versioning to support evolution of cryptocurrency protocols 