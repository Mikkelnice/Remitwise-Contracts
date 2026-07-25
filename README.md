#RemitWise Smart Contracts

Stellar Soroban smart contracts for the RemitWise remittance platform.

Overview

This workspace contains the core smart contracts that power RemitWise's post-remittance financial planning features:

- "remittance_split" (remittance_split/README.md): Automatically splits remittances into spending, savings, bills, and insurance.
- "savings_goals" (savings_goals/README.md): Goal-based savings with target dates and locked funds.
- "bill_payments" (bill_payments/README.md): Automated bill payment tracking and scheduling, including recurring bill schedule lifecycle management.
- "insurance" (insurance/README.md): Micro-insurance policy management and premium payments.
- "family_wallet" (family_wallet/README.md): Family governance, multisig approvals, spending controls, and emergency transfer controls.
- "orchestrator" (orchestrator/README.md): Cross-contract coordination and execution of end-to-end remittance flows.
- "reporting" (reporting/README.md): Financial reporting and insights.
- "emergency_killswitch" (emergency_killswitch/README.md): Centralized emergency pause controls across contracts.
- "remitwise-common" (remitwise-common/README.md): Shared types, constants, and utilities used across contracts.

Shared Components

remitwise-common

A common crate containing shared types, enums, constants, and utilities used across multiple contracts.

Shared Types

- "Category": Financial categories ("Spending", "Savings", "Bills", "Insurance").
- "FamilyRole": Access control roles ("Owner", "Admin", "Member", "Viewer").
- "CoverageType": Insurance coverage types ("Health", "Life", "Property", "Auto", "Liability").
- "EventCategory" and "EventPriority": Event logging categories and priorities.

Shared Constants

- Pagination limits ("DEFAULT_PAGE_LIMIT", "MAX_PAGE_LIMIT").
- Storage TTL values ("INSTANCE_LIFETIME_THRESHOLD", "ARCHIVE_LIFETIME_THRESHOLD", etc.).
- Contract versioning ("CONTRACT_VERSION").
- Batch operation limits ("MAX_BATCH_SIZE").

Shared Utilities

- "clamp_limit()": Validates and clamps pagination limits.
- "validate_period()": Validates the logical ordering of start and end periods.
- "same_address()": Compares two Soroban "Address" values without requiring callers to clone an address solely for equality checks.
- "RemitwiseEvents": Provides standardized event emission through "emit()" and "emit_batch()".

"same_address()" Helper

The "same_address()" helper provides a consistent way to compare two Soroban "Address" values while avoiding unnecessary address cloning at call sites.

Use the helper when comparing addresses:

if same_address(&owner, &caller) {
    // Address values match.
}

The helper is intended to:

- Improve readability of address comparisons.
- Reduce accidental ".clone()" calls used only to perform equality checks.
- Provide a shared comparison pattern across contracts.
- Preserve backwards compatibility with existing public contract APIs.

The helper does not change the semantics of address equality and does not modify or normalize either address.

Shared Enums & Constants Stability Coverage

This project includes dedicated compatibility guards in "remitwise-common" to prevent breaking changes across dependent contracts.

Coverage includes:

- Invariant tests for enum discriminants and event tags.
- Ordering assumptions verified for role and coverage definitions.
- Round-trip serialization via "soroban_sdk::IntoVal" and "TryFromVal".
- Event topic and payload serialization checks for "RemitwiseEvents".
- Pagination and TTL constant stability assertions.

Running the Compatibility Coverage

cargo test -p remitwise-common

Security Notes

- Enums use "#[repr(u32)]" and are asserted in tests, reducing the risk of contract integration drift.
- Shared constants used for pagination, batch size, storage TTL, and signature timeout are protected by stability checks.
- "clamp_limit()" includes explicit tests for overflow, underflow, and boundary conditions.
- "same_address()" is covered by tests for equal and unequal address values.

CLI Tool

A custom Rust CLI is provided for interacting with the contracts without a UI.

See "cli/README.md" (cli/README.md) for usage instructions.

Additional Components

- indexer: TypeScript event indexer for off-chain querying and analytics. See "indexer/README.md" (indexer/README.md).
- analytics: On-chain analytics and reporting.
- orchestrator: Cross-contract coordination.
- reporting: Financial reporting and insights.

Entrypoint Auth Inventory

Use the helper script below to print every public contract entrypoint alongside its detected authorization model:

python3 scripts/entrypoint_auth_inventory.py
python3 scripts/entrypoint_auth_inventory.py remittance_split

This is useful for reviewers who want a quick inventory of which entrypoints are authenticated, read-only, or otherwise permissionless.

Prerequisites

- Rust (latest stable version).
- Stellar CLI ("soroban-cli").
- Cargo.

Compatibility

Tested Versions

These contracts have been developed and tested with the following versions:

- Soroban SDK: "21.0.0"
- Soroban CLI: "21.0.0"
- Rust Toolchain: "stable" with "wasm32-unknown-unknown" and "wasm32v1-none" targets.
- Protocol Version: Compatible with Stellar Protocol 20+ (Soroban Phase 1).
- Network: Testnet and Mainnet ready.

Version Compatibility Matrix

Component| Version| Status| Notes
soroban-sdk| 21.0.0| ✅ Tested| Current stable release
soroban-cli| 21.0.0| ✅ Tested| Matches SDK version
Protocol 20| -| ✅ Compatible| Soroban Phase 1 features
Protocol 21+| -| ⚠️ Untested| Compatibility should be validated before deployment

Upgrading to New Soroban Versions

When a new Soroban SDK or protocol version is released, follow these steps to validate and upgrade.

1. Review Release Notes

Check the Soroban SDK releases for:

- Breaking changes in contract APIs.
- New features or optimizations.
- Deprecated functions.
- Protocol version requirements.

2. Update Dependencies

Update the SDK version in all applicable contract "Cargo.toml" files:

[dependencies]
soroban-sdk = "X.Y.Z"

[dev-dependencies]
soroban-sdk = { version = "X.Y.Z", features = ["testutils"] }

Contracts to update include:

- "remittance_split/Cargo.toml"
- "savings_goals/Cargo.toml"
- "bill_payments/Cargo.toml"
- "insurance/Cargo.toml"
- "family_wallet/Cargo.toml"
- "data_migration/Cargo.toml"
- "reporting/Cargo.toml"
- "orchestrator/Cargo.toml"

3. Update Soroban CLI

cargo install --locked --version X.Y.Z soroban-cli

Verify the installation:

soroban version

4. Run the Full Test Suite

cargo clean
cargo test
./scripts/run_gas_benchmarks.sh

5. Validate on Testnet

Build optimized contracts:

cargo build --release --target wasm32-unknown-unknown

Deploy a contract:

soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/<contract_name>.wasm \
  --source <your-key> \
  --network testnet

Test a contract interaction:

soroban contract invoke \
  --id <contract-id> \
  --source <your-key> \
  --network testnet \
  -- <function-name> <args>

6. Check for Breaking Changes

Common breaking changes to watch for include:

- Storage API changes, including TTL and archival patterns.
- Event emission and topic structure changes.
- Authorization and signature verification changes.
- Numeric type changes involving "i128", "u128", or fixed-point math.
- Contract lifecycle and upgrade pattern changes.

7. Update Documentation

After successful validation:

- Update this compatibility section with new versions.
- Document migration steps in "DEPLOYMENT.md".
- Update code examples if APIs change.
- Regenerate contract bindings if necessary.

Known Breaking Changes

SDK 21.0.0

No known breaking changes from the previous stable version affecting these contracts.

Future Considerations

- Protocol 21+: May introduce new storage pricing or TTL requirements.
- SDK 22.0.0+: Monitor for changes to contract storage patterns, event APIs, or authorization flows.

Installation

Install the Soroban CLI:

cargo install --locked --version 21.0.0 soroban-cli

Build all contracts:

cargo build --release --target wasm32-unknown-unknown

Examples

The workspace includes runnable examples for each contract in the "examples/" directory.

To run an example:

cargo run --example <example_name>

Contract| Example Command
Remittance Split| "cargo run --example remittance_split_example"
Savings Goals| "cargo run --example savings_goals_example"
Bill Payments| "cargo run --example bill_payments_example"
Insurance| "cargo run --example insurance_example"
Family Wallet| "cargo run --example family_wallet_example"
Reporting| "cargo run --example reporting_example"
Orchestrator| "cargo run --example orchestrator_example"

«[!NOTE]
These examples run in a mocked environment and do not require a connection to a Stellar network.»

Documentation

- "Contributor Overview" (docs/CONTRIBUTOR_OVERVIEW.md) - Onboarding guide for contributors.
- "ADR: Ban unwrap in Release Builds" (docs/adr-ban-unwrap-in-release.md) - Why "unwrap" and "panic" are forbidden in production contract code.
- "Changelog" (CHANGELOG.md) - Conventional-commits-style release history.
- "Token Decimal Catalogue" (docs/DECIMAL_CATALOGUE.md) - Reference table of decimals expected for each canonical token.
- "Authorization Matrix" (docs/AUTHORIZATION_MATRIX.md) - Per-entrypoint caller authorization requirements.
- "Pause Playbook" (docs/PAUSE_PLAYBOOK.md) - Emergency pause mechanisms and recovery procedures.
- "Committed Hashes" (docs/COMMITTED_HASHES.md) - Request-hash coverage and verification guidance.
- "Family Wallet Design" (docs/family-wallet-design.md) - Family wallet design documentation.
- "Reporting Admin Rotation" (docs/reporting-admin-rotation.md) - Two-step upgrade-admin handoff procedure.
- "Event Indexing Guide" (docs/INDEXING.md) - Mapping contract events to off-chain tables.
- "Financial Health Score Model" (docs/HEALTH_SCORE.md) - HealthScore weights, inputs, clamping, and examples.
- "Frontend Integration Notes" (docs/frontend-integration.md) - Frontend integration guidance.
- "String and Bytes Canonicalisation" (docs/CANONICALISATION.md) - Canonicalisation rules.
- "Storage Layout Reference" (STORAGE_LAYOUT.md) - Contract storage layout.
- "Contract Specs & Migrations" (docs/MIGRATIONS.md) - How to bump contract specs without breaking existing storage.
- "Event Indexer" (indexer/README.md) - Off-chain event indexing and querying.
- "Audit Trail" (docs/AUDIT_TRAIL.md) - Reconstructing historical state from events.
- "Killswitch Trust Model" (docs/killswitch-trust-model.md) - Emergency killswitch trust model.
- "Tagging Feature" (TAGGING_FEATURE.md) - Tag-based organization system.
- "Threat Model" (THREAT_MODEL.md) - Security analysis and mitigations.
- "Entrypoint Threat Breakdown" (docs/THREAT_MODEL.md) - STRIDE-style threat analysis per contract entrypoint.
- "Security Review Summary" (SECURITY_REVIEW_SUMMARY.md) - Security review findings.
- "Event Versioning ADR" (docs/events-versioning.md) - Event versioning rationale.
- "Event Versioning Discipline" (docs/EVENT_VERSIONING.md) - Backward-compatibility rules and migration guidance.

Contracts

Remittance Split

Handles automatic allocation of remittance funds into different categories.

Key Functions

- "initialize_split": Set percentage allocation for spending, savings, bills, and insurance.
- "get_split": Get the current split configuration.
- "calculate_split": Calculate actual amounts from a total remittance.

Events

- "SplitInitializedEvent": Emitted when split configuration is initialized.
  
  - "spending_percent"
  - "savings_percent"
  - "bills_percent"
  - "insurance_percent"
  - "timestamp"

- "SplitCalculatedEvent": Emitted when split amounts are calculated.
  
  - "total_amount"
  - "spending_amount"
  - "savings_amount"
  - "bills_amount"
  - "insurance_amount"
  - "timestamp"

Savings Goals

Manages goal-based savings with target dates.

Key Functions

- "create_goal": Create a new savings goal.
- "add_to_goal": Add funds to a goal.
- "get_goal": Get goal details.
- "is_goal_completed": Check whether a goal target has been reached.
- "archive_completed_goals": Archive completed goals to reduce storage.
- "get_archived_goals": Query archived goals.
- "restore_goal": Restore an archived goal.
- "cleanup_old_archives": Permanently delete old archives.
- "get_storage_stats": Get storage usage statistics.

Events

- "GoalCreatedEvent": Emitted when a new savings goal is created.
  
  - "goal_id"
  - "name"
  - "target_amount"
  - "target_date"
  - "timestamp"

- "FundsAddedEvent": Emitted when funds are added to a goal.
  
  - "goal_id"
  - "amount"
  - "new_total"
  - "timestamp"

- "GoalCompletedEvent": Emitted when a goal reaches its target amount.
  
  - "goal_id"
  - "name"
  - "final_amount"
  - "timestamp"

Bill Payments

Tracks and manages bill payments with recurring support.

Key Functions

- "create_bill": Create a new bill with an optional "external_ref".
- "pay_bill": Mark a bill as paid and create the next recurring bill when applicable.
- "set_external_ref": Owner-only update or clear operation for a bill's "external_ref".
- "get_unpaid_bills": Get all unpaid bills.
- "get_total_unpaid": Get the total amount of unpaid bills.
- "archive_paid_bills": Archive paid bills to reduce storage.
- "get_archived_bills": Query archived bills.
- "restore_bill": Restore an archived bill.
- "bulk_cleanup_bills": Permanently delete old archives.
- "get_storage_stats": Get storage usage statistics.
- "create_bill_schedule": Create a recurring or one-off bill schedule.
- "modify_bill_schedule": Modify an existing bill schedule.
- "cancel_bill_schedule": Cancel an active bill schedule.
- "execute_due_bill_schedules": Execute all due bill schedules.
- "get_bill_schedules": Get all schedules for an owner.
- "get_bill_schedule": Get a specific schedule by ID.

Events

- "BillCreatedEvent": Emitted when a new bill is created.
  
  - "bill_id"
  - "name"
  - "amount"
  - "due_date"
  - "recurring"
  - "external_ref"
  - "timestamp"

- "BillPaidEvent": Emitted when a bill is marked as paid.
  
  - "bill_id"
  - "name"
  - "amount"
  - "timestamp"

- "RecurringBillCreatedEvent": Emitted when a recurring bill generates the next bill.
  
  - "bill_id"
  - "parent_bill_id"
  - "name"
  - "amount"
  - "due_date"
  - "timestamp"

- "ScheduleCreatedEvent": Emitted when a bill schedule is created.
  
  - "schedule_id"
  - "owner"

- "ScheduleExecutedEvent": Emitted when a bill schedule is executed.
  
  - "schedule_id"

- "ScheduleModifiedEvent": Emitted when a bill schedule is modified.
  
  - "schedule_id"

- "ScheduleCancelledEvent": Emitted when a bill schedule is cancelled.
  
  - "schedule_id"

- "ScheduleMissedEvent": Emitted when a recurring schedule skips intervals.
  
  - "schedule_id"
  - "missed"

Insurance

Manages micro-insurance policies and premium payments.

Key Functions

- "create_policy": Create a new insurance policy with an optional "external_ref".
- "pay_premium": Pay a monthly premium.
- "set_external_ref": Owner-only update or clear operation for a policy's "external_ref".
- "get_active_policies": Get all active policies.
- "get_total_monthly_premium": Calculate total monthly premium cost.
- "deactivate_policy": Deactivate an insurance policy.

Bill and insurance events include "external_ref" where applicable for off-chain linking.

Events

- "PolicyCreatedEvent": Emitted when a new insurance policy is created.
  
  - "policy_id"
  - "name"
  - "coverage_type"
  - "monthly_premium"
  - "coverage_amount"
  - "external_ref"
  - "timestamp"

- "PremiumPaidEvent": Emitted when a premium is paid.
  
  - "policy_id"
  - "name"
  - "amount"
  - "next_payment_date"
  - "timestamp"

- "PolicyDeactivatedEvent": Emitted when a policy is deactivated.
  
  - "policy_id"
  - "name"
  - "timestamp"

Family Wallet

Manages family roles, spending controls, multisig approvals, and emergency transfer policies.

Key Functions

- "init": Initialize the owner, members, default multisig configuration, and emergency settings.
- "add_member": Add a family member and assign a role.
- "update_spending_limit": Update a member's spending limit.
- "configure_multisig": Configure multisig approval requirements.
- "propose_transaction": Propose a transaction requiring multisig approval.
- "sign_transaction": Sign a pending multisig transaction.
- "withdraw": Execute a direct or multisig-gated withdrawal.
- "configure_emergency": Configure emergency transfer settings.
- "set_emergency_mode": Enable or disable emergency mode.
- "propose_emergency_transfer": Propose an emergency transfer.
- "pause": Pause wallet operations.
- "unpause": Resume wallet operations.
- "set_pause_admin": Set the pause administrator.
- "archive_old_transactions": Archive old transaction records.
- "cleanup_expired_pending": Remove expired pending transactions.
- "get_storage_stats": Get storage usage statistics.

For full design details, see "docs/family-wallet-design.md" (docs/family-wallet-design.md).

Reporting

Provides summarized views of user financial data across the RemitWise ecosystem.

Key Features

- "get_remittance_summary": Aggregates split, savings, bills, and insurance data into a single summary.
- Graceful Degradation: Uses a "DataAvailability" indicator ("Complete", "Partial", or "Missing") so summary queries can return available data without hard failure when upstream contracts are unresponsive, unconfigured, or unavailable.

Orchestrator

Coordinates cross-contract operations and end-to-end remittance flows.

See "orchestrator/README.md" (orchestrator/README.md) for detailed functionality and integration guidance.

Emergency Killswitch

Provides centralized emergency pause controls across supported contracts.

See "emergency_killswitch/README.md" (emergency_killswitch/README.md) for detailed functionality and operational procedures.

Events

All contracts emit events for important state changes, enabling real-time tracking and frontend integration.

Events follow Soroban best practices and include:

- Relevant IDs: Events include the ID of the entity being acted upon where applicable.
- Amounts: Financial events include transaction amounts where applicable.
- Timestamps: Events include the ledger timestamp where applicable.
- Context Data: Additional contextual information such as names, dates, and references.

Event Topics

Each contract uses short symbol topics for efficient event identification:

- Remittance Split: "init", "calc"
- Savings Goals: "created", "added", "completed"
- Bill Payments: "created", "paid", "recurring"
- Insurance: "created", "paid", "deactive"
- Family Wallet: "added", "member", "updated", "limit", "emerg/*", "wallet/*"

Querying Events

Events can be queried from the Stellar network using the Soroban SDK or supported RPC/event-indexing infrastructure.

Each event structure is exported according to the relevant contract interface and can be decoded using the contract schema.

Testing

Run tests for all contracts:

cargo test

Issue #1141: "same_address()" Helper

The "same_address()" helper is covered by unit tests to ensure:

- Two identical addresses return "true".
- Two different addresses return "false".
- Address comparison does not require unnecessary caller-side cloning.
- Existing contract behaviour remains unchanged.

Run the shared utility tests:

cargo test -p remitwise-common

Feature Flag Consistency

The CI pipeline includes a feature flag consistency check:

python3 scripts/check_features.py

This verifies that every "cfg(feature = "...")" reference in Rust source code has a corresponding entry in the crate's "[features]" section.

Encrypted Migration Payload Decode Safety

The "data_migration" crate supports an opaque encrypted payload transport format for off-chain migration tooling.

Format

enc:v1:<base64>

Where:

- "enc:v1:" is a strict marker prefix.
- "<base64>" is the base64-encoded ciphertext blob.

Security 
