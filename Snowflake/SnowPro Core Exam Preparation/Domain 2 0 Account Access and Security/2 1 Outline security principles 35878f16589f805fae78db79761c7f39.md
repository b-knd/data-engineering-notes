# 2.1 Outline security principles

## Network securities and policies

- What is network policies
    - Control inbound access to Snowflake services and internal stage
    - By default, allow all connections
    - SECURITYADMIN can use network policy to allow/deny access based on request’s origin
- How it works - network rules + policies
    - Network policies references network rules
    - Network rules group related identifier (IP ranges, VPC endpoint IDs) into logical units that are added to policy’s allowed/blocked list
        - Presence of allowed list automatically blocks every other connection of the same type without explicit blocking
        - Blocked list takes higher precedence over allowed list (if both exists)
    - Policy precedence
        - Security integration-level → user-level → account-level
    - If user logged in when network policy applies → user cannot execute additional queries
- Where can it be applied
    - Account level - applied to all users in account
    - User level - granular control per individual user
    - Security integration level - for SCIM, Snowflake OAuth and External OAuth
- Protecting internal stages (aws)
    - To restrict access to internal stages, enable ENFORCE_NETWORK_RULES_FOR_INTERNAL_STAGES
    - Have limit on number of identifiers depending on Snowflake edition
- Private connectivity
    - Private connectivity network rules (AWS VPCE IDs or Azure LinkIDs) takes precedence over IPv4 rules
    - If want to block public access while allowing private connectivity, need two separate network rules, one to allow private endpoint, another one to block IPv4 traffic
    *simplier analogy, think of VPCE and IPv4 as separate entrance, setting only one allow the other one to bypass*
- Best practices
    - Keep network rules small and scoped (by region, user group or app) rather than large monolithic list
    - Add comments to rules for maintainability
    - Test policies at user level before applying account-wide
    - Replication of network policies across accounts is supported for failover schenarios

## Multi-Factor Authentication (MFA) enforcement

- Snowflake mandated multi-factor authentication for human users logging in with password, enforcing by default for all accounts created since 2024
- Block single-factor password sign-ins, requires user to utilise app like Google Authenticator, Authy, or Duo Mobile
- How to enforce and use MFA
    - Default policy - newly created account automatically require MFA for users, old account must enable this via authentication policies
    - Setup - user can enable MFA directly in Snowsight profile setting
    - Application - while MFA is required for human users, it does not apply to service users
    - Authentication policies - administrators can specifically mandate MFA for all human password logins unsing MFA_ENROLLMENT = ‘REQUIRED’

## Federated authentication

- Enable Single Sign-On (SSO) by delegating user authentication to external Identity Provider (IdP) like Okta, Microsoft Entra ID (Azure AD) or ADFS, typically using SAML 2.0
- Separate credential management from Snowflake access, allowing users to log in securely with corporate credentials
- Key aspects
    - SAML 2.0 Integration - Snowflake acts as service provider (SP) using SAML 2.0 to communicate with IdP, enabling secure SSO
    - IdP support - support providers include Okta, Microsoft Entra ID (formerly Azure AD), PingOne, and others
    - User management - user must exist in both IdP and Snowflake, recommended to create users with no password in Snowflake to force SAML authentication
    - Workload Identity Federation (WIF) - for services/applications, WIF allows securely authenticating to Snowflake using cloud-native identities (AWS IAM, Azure VMs, GCP) or OIDC without long-lived credentials
- Common use case flow
    - User clicks snowflake application in company portal
    - IdP authenticates the user
    - IdP sends SAML token to Snowflake
    - Snowflake verifies token and grants access
- Snowflake uses CREATE SECURITY INTEGRATION command with TYPE = SAML2 to establish connection