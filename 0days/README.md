# Zero-Day Vulnerabilities

This directory contains the findings and exploit scripts for zero-day vulnerabilities discovered in the Vuln-Bank application. These are vulnerabilities that weren't documented or previously known.

## Architecture Diagram

```mermaid
graph TD
    subgraph "Banking Operations"
        A[User] -->|Request| B[API Layer]
        B -->|Process| C[Bill Payment]
        B -->|Process| D[Card Management]
        B -->|Authenticate| E[JWT Auth]
        C -->|Update| F[Database]
        D -->|Update| F
        C -->|Generate| G[Reference Numbers]
    end
    
    subgraph "Zero-Day Vulnerabilities"
        V1[Negative Bill Payment] -.->|Exploits| C
        V2[Negative Card Limit] -.->|Exploits| D
        V3[Card Type SQL Injection] -.->|Exploits| D
        V3 -.->|Compromises| F
        V4[Reference Number Manipulation] -.->|Exploits| G
        V5[Hardcoded JWT Secret] -.->|Exploits| E
    end
```

The diagram above illustrates the banking operations in the Vuln-Bank application and how the zero-day vulnerabilities can be exploited to compromise the system.

## Vulnerabilities Overview

1. **Negative Bill Payment**: The application doesn't validate if payment amounts are negative, allowing attackers to increase their account balance by making negative payments.
2. **Negative Card Limit**: The application doesn't validate if card limits are negative, providing another method to generate unlimited funds.
3. **Card Type SQL Injection**: The application doesn't properly sanitize the `card_type` parameter, allowing SQL injection attacks.
4. **Reference Number Manipulation**: The application generates predictable reference numbers and doesn't enforce uniqueness constraints.
5. **Hardcoded JWT Secret Key**: The JWT secret key is hardcoded in the source code, allowing attackers to forge valid tokens.

## Attack Flow Diagrams

### Negative Bill Payment Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: POST /bill_payment with negative amount
    B->>D: Update balance (adds instead of subtracts)
    B->>A: Return success + increased balance
    Note over A,B: Attacker's balance increases instead of decreases
```

### Negative Card Limit Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: Create virtual card with negative limit
    B->>D: Store card with negative limit
    B->>A: Return card details
    A->>B: Make payment with card
    B->>D: Process payment (adds to balance instead of subtracting)
    B->>A: Return success + increased balance
    Note over A,B: Attacker generates unlimited funds
```

### Card Type SQL Injection Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: POST /virtual_cards with SQL injection in card_type
    B->>D: Execute malicious SQL query
    D->>B: Return sensitive data
    B->>A: Return sensitive data in response
    Note over A,D: Attacker extracts data from any table
```

### Reference Number Manipulation Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: Make legitimate payment to get reference number
    B->>D: Generate predictable reference number
    B->>A: Return reference number
    A->>A: Predict next reference number
    A->>B: Submit duplicate payment with predicted reference
    B->>D: Process payment (no uniqueness check)
    B->>A: Return success
    Note over A,B: Attacker creates accounting discrepancies
```

### Hardcoded JWT Secret Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>A: Extract hardcoded JWT secret from source code
    A->>A: Forge admin JWT token with secret
    A->>B: Request with forged admin token
    B->>B: Verify token (succeeds)
    B->>D: Query as admin
    D->>B: Return admin data
    B->>A: Return admin data
    Note over A,B: Attacker has full admin access
```

## Exploit Scripts

The following exploit scripts demonstrate how to exploit each vulnerability:

- [negative_bill_payment/exploit.py](./negative_bill_payment/exploit.py): Exploits negative bill payment vulnerability to increase account balance
- [negative_card_limit/exploit.py](./negative_card_limit/exploit.py): Exploits negative card limit vulnerability
- [card_type_sqli/exploit.py](./card_type_sqli/exploit.py): Exploits SQL injection in card_type parameter
- [reference_number_manipulation/exploit.py](./reference_number_manipulation/exploit.py): Exploits reference number manipulation vulnerability
- [hardcoded_jwt_secret_exploit.py](./hardcoded_jwt_secret_exploit.py): Exploits the hardcoded JWT secret key to forge tokens

## Remediation Recommendations

1. **Validate Payment Amounts**: Ensure payment amounts are positive and within reasonable limits.
2. **Validate Card Limits**: Ensure card limits are positive and within allowed ranges.
3. **Parameterize SQL Queries**: Use parameterized queries for all database operations.
4. **Secure Reference Number Generation**: Use cryptographically secure random numbers for reference numbers and enforce uniqueness.
5. **Secure JWT Implementation**: Use environment variables and strong secrets for JWT implementation.
