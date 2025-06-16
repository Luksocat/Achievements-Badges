LSP 27 - Achievement Badge: A Modular Reputation System Architecture

1. Introduction
This document provides a detailed technical specification for a modular and composable implementation of the LSP27 Achievement Badge standard. The architecture is designed to provide developers with maximum flexibility and gas efficiency, allowing them to select and combine features to suit their specific use case.
The core design principle is to separate distinct functionalities into individual, inheritable contracts. A minimal core contract provides the essential badge-awarding logic, including a user-consent-driven propose-and-claim mechanism. Optional extension contracts can then add further features such as individual badge metadata, on-chain recipient badge enumeration, LSP6 permission integration, and LSP1 universal notifications.
This approach results in a highly adaptable system, from a lightweight, gas-friendly badge for simple applications to a full-featured, upgradable system for complex decentralized communities and applications within the LUKSO ecosystem.
2. Purpose and Industry Applications
The primary purpose of the LSP27 standard is to create a framework for issuing non-transferable, on-chain achievement badges. These badges serve as verifiable, digital credentials that are permanently associated with a specific Universal Profile. Unlike transferable digital assets (such as LSP8 NFTs), these badges cannot be sold or exchanged, ensuring they represent the genuine accomplishments and reputation of the holder.
This modular implementation allows developers to build tailored reputation and achievement systems with verifiable, issuer-signed credentials.
Industry Applications:
Education and Professional Development:
Certifications: Educational institutions and certification bodies can issue badges for course completions, degrees, or professional accreditations. These credentials are cryptographically secure and easily verifiable by employers, mitigating fraud.
Proof of Attendance: Conferences and workshops can award badges to attendees and speakers, creating an immutable record of professional engagement and continuing education.
Gaming and Esports:
Achievements: Game developers can award badges for specific in-game accomplishments, contributing to a persistent, cross-platform gamer profile that extends beyond any single gaming environment.
Tournament Records: Organizers can award badges to tournament participants and victors, thereby building their on-chain reputation within the competitive gaming community.

Decentralized Autonomous Organizations (DAOs) and Online Communities:
Contribution and Reputation: DAOs can award badges to recognize significant contributions, such as the submission of a successful proposal, fulfillment of a specific role, or completion of a bounty.
Governance Rights: Badges can signify membership in specific working groups or tiered voting rights, enabling more nuanced governance models based on proven contribution rather than solely on token holdings.

Brand Loyalty and Marketing:
Customer Engagement: Brands can issue badges to reward their most loyal customers or engaged community members.
Gated Access: Access to exclusive content, product releases, or events can be granted to profiles holding a specific badge, creating a verifiable and automated access control system.

Personal Milestones:
Self-Sovereign Records: Applications can be developed that allow individuals to self-issue badges for personal accomplishments, creating a decentralized and user-curated record of life events and achievements.

3. Core Architecture: Composability and Extensibility
The system is built upon a foundation of abstract contracts that serve as feature modules. A base contract, LSP27AchievementBadge, implements the minimal required functionality. Various LSP27With... extension contracts can then be inherited to progressively add features.
This design offers several key advantages:
Gas Efficiency: Deployments only include the bytecode for the features that are actually used, minimizing costs.
Clarity: Separating concerns into different contracts makes the codebase easier to read, audit, and maintain.
Flexibility: Developers can easily create custom combinations of features beyond the provided presets.
Extensibility: The use of virtual functions and hooks (_afterBadgeAwarded, _afterBadgeReturned) allows new functionality to be added without modifying the core logic.

The following sections detail each component of this architecture.
4. Contract Component Analysis
4.1. LSP27AchievementBadge (Core Contract)
This contract forms the foundation of the system, providing the essential logic for an achievement badge, including critical user-consent functionality.
Function: To manage the ownership of badges in the most gas-efficient manner possible. It supports two awarding models: a direct awardBadge function for trusted scenarios and a proposeBadge/claimBadge workflow that allows users to accept or implicitly reject a badge.
Key Functions:
awardBadge(...): Directly assigns a badge to a recipient.
proposeBadge(...): Offers a badge to a recipient, who must then claim it.
claimBadge(...): Allows a recipient to accept a proposed badge, completing the award process.
returnBadge(...): Allows a holder to voluntarily return (effectively burn) their badge.
hasBadge(...): A public view function to check if an address holds a badge.
Extension Hooks: The contract includes _afterBadgeAwarded and _afterBadgeReturned as internal virtual functions. These serve as plug-in points for extensions to inject custom logic.

4.2. LSP27WithBadgeMetadata (Extension)
This essential extension provides a standardized way to associate rich metadata with each unique badgeId.
Function: To give each badge type its own identity (e.g., name, description, image) by linking its badgeId to an LSP4-compliant metadata JSON file.
Mechanism: It introduces a function setBadgeMetadata that allows the contract owner to set a metadata URI for a given badge. This information is stored in the contract's ERC725Y data store using a standardized key format: keccak256(abi.encodePacked("LSP27BadgeMetadata", badgeId)).
Indexer Workflow: This pattern enables a clear discovery process for UIs and indexers:

Listen for BadgeAwarded(badgeId, ...) events.
Use the badgeId from the event as a unique identifier.
Construct the badgeMetadataKey using the standardized formula.
Fetch the metadata URI from the contract's storage via getData(badgeMetadataKey).
Resolve the URI to retrieve and display the badge's rich metadata.

4.3. LSP27WithArrayTracking (Extension)
This module solves the common enumeration problem associated with mappings, allowing for efficient on-chain retrieval of all badges held by a user.
Function: To track and expose a user's complete collection of badges in a public array.
Mechanism: It overrides the _afterBadgeAwarded and _afterBadgeReturned hooks to manage a dynamic array of badges for each recipient. For removals, it employs the gas-efficient "swap-and-pop" method.

4.4. LSP27WithLSP6Permissions (Extension)
This module integrates the contract with LUKSO's LSP6KeyManager, enabling a more sophisticated and decentralized access control model.
Function: To delegate badge-awarding rights to addresses authorized by an LSP6KeyManager, which typically acts as the owner of the badge contract.
Mechanism: It overrides the onlyCreator modifier. The revised check queries the LSP6KeyManager to determine if the msg.sender has the required permission to award badges, enabling DAOs and other smart contract-based entities to manage permissions effectively.

4.5. LSP27WithLSP1Notifications (Extension)
This module enhances interoperability within the LUKSO ecosystem by notifying a recipient's Universal Profile upon the awarding of a badge.
Function: To inform recipient contracts, such as Universal Profiles, that a badge has been awarded to them.
Mechanism: It hooks into _afterBadgeAwarded to call the universalReceiver function on the recipient's address. The profile can then react to this notification, for instance by displaying the new badge in a user-facing application.

5. Preset Contract Configurations
To streamline deployment for common use cases, the architecture includes two preset contracts that combine several extensions into a single, ready-to-use package.
5.1. LSP27Standard
A standard, well-rounded implementation suitable for a majority of projects.
Composition: LSP27WithBadgeMetadata + LSP27WithArrayTracking + LSP27WithLSP1Notifications.
Features: Provides the core functionality (including propose/claim), individual badge metadata, on-chain badge enumeration for recipients, and LSP1 notifications for ecosystem compatibility.

5.2. LSP27Advanced
A full-featured, upgradable implementation designed for complex and long-lived applications requiring maximum security and forward-compatibility.
Composition: Includes all extensions, in addition to ReentrancyGuardUpgradeable and UUPSUpgradeable.
Features: Provides all available badge functionalities, including the core propose/claim mechanism, individual metadata, and LSP6 permissions, while also incorporating re-entrancy protection and UUPS proxy-based upgradability.

6. Implementation Source Code
The complete, commented source code for the entire modular system, including the new LSP27WithBadgeMetadata extension, is provided below for technical reference.
