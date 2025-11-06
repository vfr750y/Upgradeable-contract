# Box Upgradeable Contract

This project implements an upgradeable smart contract system using the UUPS (Universal Upgradeable Proxy Standard) pattern with OpenZeppelin's upgradeable contracts. The project includes two contract versions (`BoxV1` and `BoxV2`), deployment and upgrade scripts, and tests to verify the upgrade process. It uses Foundry as the development framework.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Contracts](#contracts)
  - [BoxV1.sol](#boxv1sol)
  - [BoxV2.sol](#boxv2sol)
- [Scripts](#scripts)
  - [DeployBox.s.sol](#deployboxssol)
  - [UpgradeBox.s.sol](#upgradeboxssol)
- [Tests](#tests)
  - [DeployAndUpgradeTest.t.sol](#deployandupgradetesttsol)
- [Configuration](#configuration)
  - [foundry.toml](#foundrytoml)
- [Installation](#installation)
- [Usage](#usage)
  - [Deploying the Contract](#deploying-the-contract)
  - [Upgrading the Contract](#upgrading-the-contract)
  - [Running Tests](#running-tests)
- [Dependencies](#dependencies)
- [Security Considerations](#security-considerations)
- [License](#license)

## Overview

The `Box` contract is an upgradeable smart contract that demonstrates the UUPS proxy pattern. The initial version (`BoxV1`) allows reading a stored number and supports upgrading to future versions. The upgraded version (`BoxV2`) adds functionality to set the stored number. The project includes scripts for deploying the proxy and upgrading the implementation, along with tests to verify the upgrade process.

## Prerequisites

To work with this project, you need the following tools installed:

- [Foundry](https://github.com/foundry-rs/foundry): A smart contract development toolchain.
- [Git](https://git-scm.com/): For cloning the repository and managing dependencies.
- [Node.js](https://nodejs.org/): For managing dependencies (optional if using Foundry's dependency management).
- A compatible Ethereum development environment (e.g., local node, testnet, or mainnet).

## Project Structure

```
.
├── src/
│   ├── BoxV1.sol          # Initial version of the contract
│   └── BoxV2.sol          # Upgraded version of the contract
├── script/
│   ├── DeployBox.s.sol    # Deployment script for BoxV1 with proxy
│   └── UpgradeBox.s.sol   # Upgrade script to BoxV2
├── test/
│   └── DeployAndUpgradeTest.t.sol  # Tests for deployment and upgrade
├── foundry.toml           # Foundry configuration file
└── README.md              # Project documentation
```

## Contracts

### BoxV1.sol

The initial version of the contract, `BoxV1`, is an upgradeable contract that:

- Inherits from OpenZeppelin's `Initializable`, `UUPSUpgradeable`, and `OwnableUpgradeable` contracts.
- Stores a `number` variable (default initialized to 0).
- Provides a `getNumber()` function to read the stored number.
- Includes a `version()` function that returns `1` to indicate the contract version.
- Implements `_authorizeUpgrade` (required by UUPS) to control upgrades, restricted to the contract owner.

**Key Features:**
- Uses the UUPS proxy pattern for upgrades.
- Ownable, restricting upgrades to the contract owner.
- Minimal functionality to demonstrate the upgrade process.

**Note:** The `initialize` function is commented out in the provided code. In a production environment, ensure it is implemented to properly initialize the contract's state.

### BoxV2.sol

The upgraded version of the contract, `BoxV2`, extends `BoxV1` and:

- Inherits from `UUPSUpgradeable`.
- Adds a `setNumber(uint256)` function to modify the stored `number`.
- Retains the `getNumber()` function.
- Updates the `version()` function to return `2`.
- Implements `_authorizeUpgrade` for upgrade control.

**Key Features:**
- Introduces state-modifying functionality.
- Maintains compatibility with the proxy's storage layout.

## Scripts

### DeployBox.s.sol

This script deploys the `BoxV1` contract behind an `ERC1967Proxy`:

- Deploys the `BoxV1` implementation contract.
- Deploys an `ERC1967Proxy` pointing to the `BoxV1` implementation.
- Returns the proxy address for interaction.

### UpgradeBox.s.sol

This script upgrades the proxy to use the `BoxV2` implementation:

- Uses the `DevOpsTools` library to retrieve the most recently deployed proxy address.
- Deploys the `BoxV2` implementation contract.
- Calls `upgradeTo` on the proxy to point to the new implementation.
- Returns the proxy address.

## Tests

### DeployAndUpgradeTest.t.sol

This test suite verifies the deployment and upgrade process:

- **Setup:** Deploys the proxy with `BoxV1` using the `DeployBox` script.
- **Test: `testProxyStartsAsBoxV1`:** Ensures the proxy starts with `BoxV1` functionality (e.g., `setNumber` is not available and reverts).
- **Test: `testUpgrades`:** Upgrades the proxy to `BoxV2`, verifies the version is `2`, and tests the new `setNumber` functionality.

## Configuration

### foundry.toml

The `foundry.toml` file configures the Foundry environment:

- **Source Directory:** `src`
- **Output Directory:** `out`
- **Libraries:** `lib`
- **Remappings:** Maps OpenZeppelin and Foundry DevOps dependencies for easy imports.

## Installation

1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Install Foundry:**
   If Foundry is not installed, follow the instructions at [Foundry's documentation](https://github.com/foundry-rs/foundry#installation):
   ```bash
   curl -L https://foundry.paradigm.xyz | bash
   foundryup
   ```

3. **Install Dependencies:**
   Foundry manages dependencies via Git submodules. Install them with:
   ```bash
   forge install
   ```

   The project depends on:
   - `@openzeppelin-contracts-upgradeable`
   - `@openzeppelin/contracts`
   - `foundry-devops`

4. **Verify Dependencies:**
   Ensure the `lib` directory contains the required libraries, as specified in `foundry.toml`.

## Usage

### Deploying the Contract

To deploy the `BoxV1` contract with a proxy:

```bash
forge script script/DeployBox.s.sol:DeployBox --rpc-url <RPC_URL> --private-key <PRIVATE_KEY> --broadcast
```

Replace `<RPC_URL>` with your Ethereum node URL (e.g., local, Infura, Alchemy) and `<PRIVATE_KEY>` with the deployer's private key.

This will:
- Deploy the `BoxV1` implementation.
- Deploy an `ERC1967Proxy` pointing to `BoxV1`.
- Output the proxy address.

### Upgrading the Contract

To upgrade the proxy to `BoxV2`:

```bash
forge script script/UpgradeBox.s.sol:UpgradeBox --rpc-url <RPC_URL> --private-key <PRIVATE_KEY> --broadcast
```

This will:
- Deploy the `BoxV2` implementation.
- Upgrade the proxy to point to `BoxV2`.
- Output the proxy address (unchanged).

### Running Tests

To run the test suite:

```bash
forge test
```

For verbose output (to see test logs):

```bash
forge test -vv
```

The tests verify:
- The proxy starts with `BoxV1` functionality.
- The upgrade to `BoxV2` succeeds and introduces new functionality.

## Dependencies

- **OpenZeppelin Contracts Upgradeable:** Provides `Initializable`, `UUPSUpgradeable`, `OwnableUpgradeable`, and `ERC1967Proxy`.
- **Foundry DevOps:** Provides utilities for retrieving the most recent deployment.
- **Forge Standard Library (`forge-std`):** Provides testing and scripting utilities.

These are managed as Git submodules in the `lib` directory and remapped in `foundry.toml`.

## Security Considerations

- **Initializer Safety:** Ensure the `initialize` function in `BoxV1` is implemented and called correctly during deployment to prevent reinitialization attacks. The commented-out constructor and initializer in `BoxV1.sol` should be uncommented and tested.
- **Storage Layout:** Ensure `BoxV2` maintains the same storage layout as `BoxV1` to avoid storage collisions. Any new variables in `BoxV2` should be appended to the existing layout.
- **Access Control:** The `_authorizeUpgrade` function in both contracts is currently empty, allowing any owner to upgrade. In production, add access control logic (e.g., multisig, timelock) to secure upgrades.
- **Proxy Security:** Use OpenZeppelin's `ERC1967Proxy` to ensure compliance with the UUPS standard.
- **Testing:** Thoroughly test upgrades on a testnet or local fork before mainnet deployment to verify storage and functionality.

## License

This project is licensed under the MIT License. See the `SPDX-License-Identifier: MIT` in each file for details.

---

This README provides a comprehensive guide to the project, covering its structure, usage, and key considerations. Let me know if you need further clarification or additional details!