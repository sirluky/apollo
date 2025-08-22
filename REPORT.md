# Muun Wallet Security Audit Report

## Executive Summary

After conducting a comprehensive analysis of the Muun Bitcoin Lightning wallet source code, I have evaluated their claims of being a non-custodial 2-of-2 multisig wallet. This report provides detailed findings on the security architecture, key management, Lightning Network integration, and potential risks.

**Overall Assessment: PARTIALLY NON-CUSTODIAL WITH SIGNIFICANT CAVEATS**

While Muun implements several security best practices and maintains user control over private keys, there are important trust assumptions and dependencies on Muun's servers that limit the "non-custodial" claim.

## Key Findings Summary

### ✅ Strengths
- **Proper cryptographic implementation**: Uses industry-standard Bitcoin libraries (btcsuite, BitcoinJ)
- **Client-side key generation**: Private keys are generated on the user's device
- **HD wallet implementation**: Follows BIP-32 standards with proper entropy
- **MuSig2 multisig**: Modern Schnorr signature aggregation for 2-of-2 multisig
- **Emergency recovery system**: Users can recover funds independently via emergency kit
- **Android secure storage**: Uses Android Keystore for key protection
- **Open source**: Full source code available for audit

### ⚠️ Concerns & Limitations
- **Server dependency for Lightning**: Lightning payments require Muun server cooperation
- **Submarine swap trust model**: Users must trust Muun's submarine swap implementation
- **Limited Lightning features**: No direct channel management or true Lightning routing
- **Server-assisted transaction creation**: Muun server helps construct transactions
- **Synchronized key management**: Server holds second key in 2-of-2 setup

### ❌ Critical Issues
- **Lightning Network centralization**: All Lightning functionality routes through Muun
- **Server availability dependency**: Wallet functionality significantly degraded without server
- **Submarine swap counterparty risk**: Users exposed to Muun's operational risks
- **Limited user sovereignty**: Cannot easily migrate to other Lightning implementations

## Detailed Technical Analysis

### 1. Key Management Architecture

#### Key Generation & Storage
- **Private key generation**: Done entirely client-side using cryptographically secure random number generation
- **HD wallet paths**: Uses custom schema `m/schema:1'/recovery:1'` with proper hardened derivation
- **Key encryption**: Private keys encrypted using user PIN/password before storage
- **Android Keystore**: Leverages hardware-backed security when available
- **Emergency kit**: Encrypted keys stored in PDF format for offline recovery

**Security Rating: STRONG** ✅

#### Multisig Implementation
```java
// Schema derivation paths
BASE_PATH = "m/schema:1'/recovery:1'"
CHANGE = BASE_PATH + "/change:0"
EXTERNAL = BASE_PATH + "/external:1"  
CONTACTS = BASE_PATH + "/contacts:2"
METADATA = BASE_PATH + "/metadata:3"
```

- **2-of-2 multisig**: User controls one key, Muun server holds the other
- **MuSig2 protocol**: Modern implementation using Schnorr signatures
- **Key aggregation**: Efficient single signature output despite being multisig
- **Taproot support**: Advanced spending conditions using tapscript

**Security Rating: GOOD WITH CAVEATS** ⚠️

**Caveat**: While technically 2-of-2 multisig, users depend on Muun's cooperation for normal wallet operations.

### 2. Lightning Network Integration

#### Submarine Swap Architecture
```go
type SubmarineSwap struct {
    Invoice       string
    Receiver      SubmarineSwapReceiver  
    FundingOutput SubmarineSwapFundingOutput
    PreimageInHex string
}
```

Muun doesn't manage Lightning channels directly. Instead, they use submarine swaps:
- **On-chain to Lightning**: User sends Bitcoin to Muun-controlled address, Muun pays Lightning invoice
- **Lightning to on-chain**: Muun receives Lightning payment, sends on-chain Bitcoin to user
- **Trust requirement**: Users must trust Muun to honor the swap agreements

**Security Rating: CONCERNING** ❌

**Major concerns**:
1. **Counterparty risk**: Users expose funds to Muun during swap process
2. **Centralized routing**: All Lightning payments flow through Muun
3. **Service dependency**: Lightning functionality breaks if Muun discontinues service
4. **Limited transparency**: Swap fees and routing not fully transparent

### 3. Transaction Signing Process

#### Signing Expectations Verification
```go
func (p *PartiallySignedTransaction) Verify(expectations *SigningExpectations, 
    userPublicKey *HDPublicKey, muunPublickKey *HDPublicKey) error {
    // Verify destination address
    // Verify amounts and fees  
    // Verify change address derivation
    // Prevent fee overpayment
}
```

- **User authorization required**: Every transaction requires user's private key signature
- **Transaction verification**: System validates destination, amounts, and fees before signing
- **Change address validation**: Ensures change returns to user-controlled addresses
- **Fee protection**: Prevents excessive fee payments

**Security Rating: STRONG** ✅

### 4. Emergency Recovery System

#### Recovery Kit Components
```go
type EKInput struct {
    FirstEncryptedKey  string  // User's encrypted private key
    FirstFingerprint   string  // User key fingerprint
    SecondEncryptedKey string  // Muun's encrypted private key  
    SecondFingerprint  string  // Muun key fingerprint
    RcChecksum         string  // Recovery code checksum
}
```

- **Independent recovery**: Users can recover funds without Muun's servers
- **Encrypted private keys**: Both user and Muun keys included in emergency kit
- **Output descriptors**: Standard Bitcoin descriptors for wallet scanning
- **Recovery tool**: Separate open-source tool for emergency fund recovery

**Security Rating: EXCELLENT** ✅

This is Muun's strongest "non-custodial" feature - users can definitely recover their funds independently.

### 5. Network Communication Analysis

#### Data Transmitted to Servers
Based on code analysis, Muun servers receive:
- **Public keys and addresses**: For transaction construction
- **Transaction metadata**: Amounts, destinations, timing
- **Fee preferences**: User's fee rate selections  
- **Device information**: For sync and session management
- **Usage analytics**: Performance and error reporting

**Private keys are never transmitted to servers** ✅

**Security Rating: ACCEPTABLE** ⚠️

### 6. Android Security Implementation

#### Secure Storage Provider
```java
@Singleton
public class SecureStorageProvider {
    private final KeyStoreProvider keyStore;
    private final SecureStoragePreferences preferences;
}
```

- **Android Keystore**: Hardware-backed security when available
- **Encrypted preferences**: Sensitive data encrypted before storage
- **Key wrapping**: Additional encryption layer for private keys
- **Screen protection**: Prevents screenshots of sensitive information

**Security Rating: STRONG** ✅

## Risk Assessment

### High Risk Issues

1. **Lightning Centralization Risk**
   - **Impact**: Complete loss of Lightning functionality if Muun shuts down
   - **Probability**: Low-Medium (business sustainability concerns)
   - **Mitigation**: Users can recover on-chain funds via emergency kit

2. **Submarine Swap Counterparty Risk** 
   - **Impact**: Loss of funds during swap process
   - **Probability**: Low (Muun has financial incentive to be honest)
   - **Mitigation**: Swap amounts are typically small for Lightning usage

3. **Server Dependency Risk**
   - **Impact**: Degraded wallet functionality without server access
   - **Probability**: Medium (network issues, server maintenance)
   - **Mitigation**: Core fund access still possible via emergency recovery

### Medium Risk Issues

1. **Key Management Complexity**
   - Users must properly secure emergency kits and recovery codes
   - Loss of recovery materials could mean permanent fund loss

2. **Limited Lightning Network Features**
   - Cannot switch to other Lightning wallets easily
   - No direct channel management capabilities

3. **Fee Transparency** 
   - Submarine swap fees not fully transparent to users
   - Potential for hidden costs in Lightning operations

### Low Risk Issues

1. **Code Complexity**
   - Large codebase increases potential for bugs
   - Multiple programming languages (Java, Kotlin, Go)

2. **Android Security Dependency**
   - Relies on Android Keystore implementation
   - Varies by device and Android version

## Comparison: Non-Custodial Claims vs Reality

### What IS Non-Custodial:
✅ **On-chain Bitcoin storage**: Users control private keys for on-chain funds
✅ **Emergency recovery**: Can recover all funds without Muun's cooperation  
✅ **Transaction signing**: User authorization required for all transactions
✅ **Key generation**: Private keys created client-side with proper entropy

### What is NOT Fully Non-Custodial:
❌ **Lightning Network functionality**: Requires trust and cooperation with Muun
❌ **Submarine swaps**: Users must trust Muun during swap process
❌ **Server-assisted operations**: Many wallet functions require Muun's servers
❌ **Lightning routing**: All Lightning payments flow through Muun's infrastructure

## Recommendations

### For Users

#### ✅ Safe to use for:
- **Long-term Bitcoin storage**: Emergency recovery system is robust
- **Small Lightning transactions**: Risk is limited by transaction amounts
- **Learning Lightning Network**: Good introduction to Lightning concepts

#### ⚠️ Use with caution for:
- **Large Lightning payments**: Higher counterparty risk exposure
- **Critical Lightning operations**: Service dependency creates availability risk
- **Privacy-sensitive transactions**: Muun servers see transaction metadata

#### ❌ Not recommended for:
- **Users requiring full sovereignty**: Traditional Lightning node would be better
- **Large-scale Lightning operations**: Counterparty risk too significant
- **Users in restrictive jurisdictions**: Centralized service could be blocked

### For Muun

1. **Improve transparency**: Provide clearer documentation of trust assumptions
2. **Enhance fee disclosure**: Show all costs associated with submarine swaps
3. **Consider decentralization**: Explore ways to reduce server dependencies
4. **Emergency kit improvements**: Make recovery process more user-friendly

## Conclusion

Muun Wallet represents a **hybrid custodial/non-custodial approach** rather than being purely non-custodial. While users maintain strong control over their on-chain Bitcoin funds and can recover them independently, the Lightning Network functionality introduces significant trust assumptions and server dependencies.

**For small amounts and long-term holding**, Muun provides a reasonable balance between usability and security. **For large amounts or users requiring full sovereignty**, a more traditional self-custodial solution would be preferable.

The emergency recovery system is genuinely impressive and does provide true non-custodial fund recovery, which is Muun's strongest security feature. However, users should understand that normal wallet operations, especially Lightning functionality, require ongoing trust in and cooperation from Muun's infrastructure.

**Final Verdict**: Muun is honest about being "non-custodial" in the sense that users can always recover their funds, but the Lightning implementation requires trust assumptions that limit the sovereignty typically associated with true non-custodial solutions.

---

*Audit conducted through comprehensive source code analysis of the open-source Muun wallet repository. This assessment is based on the code as of the audit date and assumes honest implementation by the Muun team.*