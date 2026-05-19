# 2.2 Define the entities and roles that are used in Snowflake

## Overview of Access Control

### Access Control Framework

- Combines RBAC and DAC to secure data, requiring every action to be performed via designated role
- Use hierarchical structure where privileges (like SELECT, INSERT) are granted to roles, and roles assigned to users, enabling strict, scalable and auditable data access management
- Key component
    - Securable objects - any entity in Snowflake to which access can be granted or denied
    - Roles - entities to which privileges are assigned, roles can be assigned to users or other roles, creating hierarchy
    - Privileges - defined levels of access to object, such as SELECT, INSERT, UPDATE or OWNERSHIP
    - Users - unique identities (people or services) that adopt roles to perform actions
- Modal
- RBAC - permission assigned to roles rather than individuals, simplifying security management based on job function
- DAC - every object has owner that can grant access to other roles

## Outline how privileges can be granted and revoked

- In Snowflake, privileges are granted and revoked to roles - not directly to users
- Typically managed by SECURITYADMIN/USERADMIN to uphold principle of least privilege
- Granting privileges (GRANT)
    - Privileges assigned to roles, allowing them to perform specific actions on objects

```sql
# grant privilege on specific object to role
GRANT SELECT ON TABLE my_table TO ROLE analyst

# grant privileges on all future object of specific type in schema
GRANT SELECT ON FUTURE TABLES IN SCHEMA myschema TO ROLE analyst

# role hierarchy
GRANT ROLE role_a TO ROLE role_b
```

- Revoking privileges (REVOKE)
    - Removes previously granted privileges or roles

```sql
# revoke specific privileges
REVOKE SELECT ON TABLE mytable FROM ROLE analyst

# revoke future grants
REVOKE SELECT ON FUTURE TABLES IN SCHEMA myschema FROM ROLE analyst

# revoke roles
REVOKE ROLE role_a FROM ROLE role_b
```

**Key considerations**

- Managed access schemas - using WITH MAANGED ACCESS centralises grant management for objects to schema owner, preventing accidental permission drift
- MANAGE GRANTS privilege - allow role (like securityadmin) to manage grants on any object, even if they don’t own it
- Ownership - owner of object has all privileges on it, which can be transferred using GRANT OWNERSHIP
- Database role - privileges on object can be granted to database roles, which can then be granted to account roles for localised management

## Explain role hierarchy and privilege inheritance

- Role can be granted to another roles, creating hierarchical structure
- Hierarchy enable privilege inheritance, where higher role automatically inherits privileges granted to lower/child roles

**System-Defined Roles**

*Cannot be dropped*

- ACCOUNTADMIN - most powerful role, manage account-level parameters, views billing and credit data and operate on all objects in account
- ORGADMIN - manages operations at organisation level
- SECURITYADMIN - manage object grant globally and creating, monitoring, and managing users and roles (inherits privileges of USERADMIN role)
- USERADMIN - dedicated to user and role management
- SYSADMIN - create warehouse, databases and objects within account (day-to-day)
- PUBLIC - pseudo-role with minimal privileges

**Custom roles**

- Custom functional roles aligned with specific business units or job functions
- Follows principle of least privilege

**Management and Auditing** 

- Granting and revoking: GRANT/REVOKE
- SHOW ROLES to view available roles and assignments
- GRANT_TO_ROLES in ACCOUNT_USAGE to audit specific access control privileges