# Authentication and Authorization Vulnerabilities

This directory contains the findings and exploit scripts for authentication and authorization vulnerabilities discovered in the Vuln-Bank application.

## Architecture Diagram

```mermaid
graph TD
    subgraph "Authentication Flow"
        A[User] -->|Credentials| B[Login Endpoint]
        B -->|SQL Query| C[Database]
        C -->|User Data| B
        B -->|Generate| D[JWT Token]
        D -->|Return| A
        A -->|Token| E[Protected Endpoints]
        E -->|Verify| F[JWT Verification]
        F -->|Access| G[Resources]
    end
    
    subgraph "Vulnerabilities"
        V1[SQL Injection] -.->|Exploits| B
        V2[JWT Implementation Flaws] -.->|Exploits| D
        V2 -.->|Exploits| F
        V3[Weak Password Reset] -.->|Exploits| H[Password Reset]
        V4[BOLA] -.->|Exploits| G
        V5[Mass Assignment] -.->|Exploits| I[User Update]
    end
    
    H -->|Reset| C
    I -->|Update| C
```

The diagram above illustrates the authentication flow in the Vuln-Bank application and how different vulnerabilities can be exploited to bypass security controls.

## Vulnerabilities Overview

1. **SQL Injection in Login**: The login functionality is vulnerable to SQL injection, allowing complete authentication bypass.
2. **JWT Implementation Flaws**: Multiple issues with JWT implementation including weak secret key, vulnerable algorithm selection, and no token expiration.
3. **Weak Password Reset Mechanism**: The password reset functionality uses a 3-digit PIN, making it easily brute-forceable.
4. **Broken Object Level Authorization (BOLA)**: The application fails to verify if the authenticated user has permission to access resources.
5. **Mass Assignment**: The application blindly binds client-provided data to internal objects without proper filtering.

## Attack Flow Diagrams

### SQL Injection Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Login Endpoint
    participant D as Database
    
    A->>B: Send malicious credentials (admin' --)
    B->>D: Execute vulnerable SQL query
    Note right of D: WHERE username='admin' --' AND password='anything'
    D->>B: Return admin user data (password check bypassed)
    B->>A: Generate and return valid JWT token
    Note over A,B: Attacker now has admin access
```

### JWT None Algorithm Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    
    A->>A: Obtain a valid JWT token
    A->>A: Decode token without verification
    A->>A: Modify payload (set is_admin=true)
    A->>A: Change algorithm to 'none'
    A->>A: Create new token with empty signature
    A->>B: Request with forged admin token
    B->>B: Verify token (succeeds due to 'none' alg)
    B->>A: Return admin data/access
    Note over A,B: Attacker has admin privileges
```

### Password Reset Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Request password reset for target user
    B->>D: Generate 3-digit PIN (100-999)
    B->>A: Confirm reset email sent
    loop Brute Force (max 900 attempts)
        A->>B: Try PIN + new password
        B->>D: Check if PIN is correct
    end
    B->>D: Update user's password
    B->>A: Password reset successful
    A->>B: Login with new password
    B->>A: Authentication successful
    Note over A,B: Attacker has taken over account
```

### BOLA Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate as regular user
    B->>A: Return valid JWT token
    A->>B: Request account balance with admin's account number
    Note right of B: No authorization check performed
    B->>D: Query admin's account data
    D->>B: Return admin's account data
    B->>A: Display admin's account balance
    Note over A,B: Attacker accessed unauthorized data
```

### Mass Assignment Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate as regular user
    B->>A: Return valid JWT token
    A->>B: Send profile update with is_admin=true
    Note right of B: No filtering of sensitive fields
    B->>D: Update user record with all fields
    D->>B: Confirm update
    B->>A: Return success
    A->>B: Access admin functionality
    B->>A: Return admin data (privilege escalation successful)
    Note over A,B: Attacker has elevated privileges
```

## Exploit Scripts

The following exploit scripts demonstrate how to exploit each vulnerability:

- [sql_injection_login.py](./sql_injection_login.py): Exploits SQL injection in login functionality to bypass authentication
- [jwt_none_algorithm.py](./jwt_none_algorithm.py): Exploits the 'none' algorithm vulnerability in JWT implementation
- [jwt_weak_secret.py](./jwt_weak_secret.py): Exploits the weak JWT secret key to forge admin tokens
- [password_reset_bruteforce.py](./password_reset_bruteforce.py): Brute forces the weak password reset PIN
- [bola_exploit.py](./bola_exploit.py): Exploits Broken Object Level Authorization to access another user's account information
- [mass_assignment_exploit.py](./mass_assignment_exploit.py): Exploits mass assignment vulnerability to elevate privileges

## Remediation Recommendations

1. **SQL Injection**: Replace string interpolation with parameterized queries.
2. **JWT Implementation**: Use a strong, randomly generated secret key, enforce token expiration, and remove the 'none' algorithm.
3. **Password Reset**: Use longer, more complex reset tokens with expiration times and implement rate limiting.
4. **Authorization**: Implement proper object-level authorization checks for all resources.
5. **Mass Assignment**: Use a whitelist of allowed properties for each endpoint.
