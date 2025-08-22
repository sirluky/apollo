# Phase 2: Build System & Dependencies - Results

## Overview
This phase analyzed the build system and dependency chains to understand external dependencies and potential security risks in the build process.

## Go libwallet Build Results ✅

### Build Status: SUCCESSFUL
- **Module**: github.com/muun/libwallet
- **Go Version**: 1.24.6 (compatible with required Go 1.22+)
- **Build Output**: All packages compiled successfully with no errors

### Key Dependencies Analyzed:

#### Bitcoin Core Libraries:
- **github.com/btcsuite/btcd v0.24.2-beta**: Core Bitcoin protocol implementation
- **github.com/btcsuite/btcd/btcec/v2 v2.3.3**: Elliptic curve cryptography
- **github.com/btcsuite/btcd/btcutil v1.1.5**: Bitcoin utility functions
- **github.com/decred/dcrd/dcrec/secp256k1/v4 v4.3.0**: secp256k1 curve implementation

#### Lightning Network:
- **github.com/lightningnetwork/lnd v0.18.0-beta**: Lightning Network daemon
- **github.com/lightningnetwork/lightning-onion**: Onion routing for Lightning
- **github.com/fiatjaf/go-lnurl v1.13.1**: LNURL protocol support

#### Cryptographic Libraries:
- **golang.org/x/crypto v0.25.0**: Go's extended cryptography package
- **github.com/btcsuite/btcwallet**: Bitcoin wallet functionality

#### Database & Storage:
- **github.com/jinzhu/gorm v1.9.16**: ORM for database operations
- **go.etcd.io/bbolt v1.3.7**: Embedded key-value database
- **modernc.org/sqlite v1.29.8**: SQLite database driver

### Security Assessment - Go Dependencies:
✅ **POSITIVE INDICATORS:**
- Uses well-established Bitcoin libraries (btcsuite)
- Standard cryptographic libraries from Go ecosystem
- Lightning Network integration through official LND libraries
- No unusual or suspicious dependencies identified

⚠️ **AREAS OF CONCERN:**
- Large dependency tree (180+ total dependencies)
- Beta version of LND (0.18.0-beta) - production should use stable versions
- Multiple database systems could increase attack surface

## Java/Android Build Analysis

### Build Status: ❌ BLOCKED (Network Restrictions)
- **Error**: Cannot resolve Android Gradle Plugin and SQLDelight dependencies
- **Blocked Domains**: dl.google.com, jitpack.io
- **Impact**: Unable to perform full build analysis

### Dependencies Analysis from build.gradle:

#### Core Java Dependencies (common module):
- **javax.validation:validation-api:1.1.0.Final**: Input validation
- **org.hibernate:hibernate-validator:5.4.2.Final**: Validation framework
- **com.fasterxml.jackson.core**: JSON processing libraries
- **com.squareup.retrofit2:retrofit:2.5.0**: HTTP client for API calls
- **io.reactivex:rxjava:1.3.8**: Reactive programming

#### Bitcoin Java Libraries:
- **com.github.muun:bitcoinj:0.15.4-taproot**: Custom fork of BitcoinJ with Taproot support
- **org.bouncycastle:bcprov-jdk15to18:1.63**: Cryptographic provider
- **com.lambdaworks:scrypt:1.4.0**: Scrypt password hashing

#### Android Framework:
- **com.android.tools.build:gradle:8.5.2**: Android build system
- **com.github.muun.sqldelight:gradle-plugin**: Custom SQLDelight fork

### Security Assessment - Java Dependencies:

✅ **POSITIVE INDICATORS:**
- Uses standard industry libraries (Jackson, Retrofit, BouncyCastle)
- Hibernate validation for input sanitization
- Custom BitcoinJ fork with Taproot support (modern Bitcoin features)
- Reproducible builds configuration present

⚠️ **POTENTIAL RISKS:**
- Custom forks of SQLDelight and BitcoinJ - need source code verification
- Older versions of some libraries (Retrofit 2.5.0 from 2018)
- JitPack dependency source introduces supply chain risk
- Unable to verify exact dependency tree due to build failure

## Build System Security Analysis

### Gradle Configuration:
- **Reproducible Builds**: ✅ Configured with `preserveFileTimestamps = false`
- **Maven Repositories**: Uses standard repositories (Google, Maven Central) plus JitPack
- **Version Pinning**: ✅ Specific versions pinned for critical dependencies

### Supply Chain Security:

#### Strengths:
- Go module with `go.sum` checksums for dependency verification
- Gradle dependency version locking
- Standard repository sources for most dependencies

#### Weaknesses:
- JitPack repository for custom forks introduces trust dependencies
- Cannot verify build artifacts match source without successful build
- Multiple external repositories increase attack surface

## Dependency Risk Assessment

### HIGH TRUST (Standard Libraries):
- btcsuite Bitcoin libraries
- Go standard library extensions
- BouncyCastle cryptography
- Android Framework components

### MEDIUM TRUST (Established but Older):
- Jackson JSON libraries (2.9.x series)
- Retrofit HTTP client (2.5.0)
- RxJava reactive streams

### REQUIRES VERIFICATION (Custom Forks):
- com.github.muun:bitcoinj:0.15.4-taproot
- com.github.muun.sqldelight:gradle-plugin
- github.com/muun/mobile (Go mobile replacement)

## Recommendations for Phase 3:

1. **Priority**: Analyze the custom BitcoinJ fork for security modifications
2. **Focus**: Review cryptographic library usage patterns in source code
3. **Investigate**: How custom forks differ from upstream versions
4. **Verify**: Proper entropy sources and key generation mechanisms

## Status: ✅ PARTIALLY COMPLETE
- Go build analysis: Complete
- Java/Android dependency analysis: Blocked by network restrictions but build files analyzed
- Supply chain security assessment: Complete based on available information