# Data Security Vulnerabilities

This directory contains the findings and exploit scripts for data security vulnerabilities discovered in the Vuln-Bank application.

## Architecture Diagram

```mermaid
graph TD
    subgraph "Data Flow"
        A[User] -->|Request| B[API Endpoints]
        B -->|Query| C[Database]
        C -->|Data| B
        B -->|Response| A
        D[Error Handler] -->|Error Messages| A
        B -->|Logs| E[Debug Information]
    end
    
    subgraph "Vulnerabilities"
        V1[Plaintext Password Storage] -.->|Affects| C
        V2[Sensitive Data Exposure] -.->|Affects| B
        V3[SQL Injection Points] -.->|Affects| B
        V3 -.->|Compromises| C
        V4[Debug Information Exposure] -.->|Affects| E
        V5[Detailed Error Messages] -.->|Affects| D
    end
```

The diagram above illustrates the data flow in the Vuln-Bank application and how different vulnerabilities can lead to data security breaches.

## Vulnerabilities Overview

1. **Plaintext Password Storage**: The application stores user passwords in plaintext, allowing attackers to directly access passwords if they gain access to the database.
2. **Sensitive Data Exposure**: The application exposes sensitive data in API responses, providing attackers with valuable information.
3. **SQL Injection Points**: Multiple SQL injection points were identified that could lead to unauthorized access to sensitive data.
4. **Debug Information Exposure**: The application exposes detailed debug information in production environments.
5. **Detailed Error Messages**: The application returns detailed error messages that reveal sensitive information.

## Attack Flow Diagrams

### Plaintext Password Storage Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: Exploit SQL injection in search endpoint
    B->>D: Execute malicious SQL query
    Note right of D: UNION SELECT id, username, password, email FROM users
    D->>B: Return plaintext passwords
    B->>A: Return plaintext passwords in response
    Note over A,B: Attacker has all user passwords
```

### Sensitive Data Exposure Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: Request profile data
    B->>D: Query user data
    D->>B: Return complete user record
    B->>A: Return all user data (including sensitive fields)
    Note over A,B: Attacker obtains excessive sensitive information
```

### SQL Injection Attack Flow

```mermaid
graph TD
    A[Attacker] -->|1. Authenticate| B[Vuln-Bank API]
    B -->|2. Return JWT token| A
    A -->|3. Send malicious search query| B
    B -->|4. Execute vulnerable SQL query| C[Database]
    C -->|5. Return data from users table| B
    B -->|6. Return sensitive data| A
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#bfb,stroke:#333,stroke-width:2px
```

### Debug Information Exposure Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    
    A->>B: Send invalid request to trigger error
    B->>B: Generate detailed error with stack trace
    B->>A: Return detailed debug information
    A->>A: Analyze stack trace for internal paths
    A->>A: Extract database structure from error
    A->>A: Map application architecture
    Note over A,B: Attacker gains detailed system insights
```

### Detailed Error Messages Attack Flow

```mermaid
sequenceDiagram
    participant A as Attacker
    participant B as Vuln-Bank API
    participant D as Database
    
    A->>B: Authenticate (get token)
    B->>A: Return JWT token
    A->>B: Send invalid data to trigger error
    B->>D: Attempt database operation
    D->>B: Return database error
    B->>A: Return detailed error message
    Note over A,B: Error contains table names, column names, SQL syntax
    A->>A: Extract database schema information
    A->>A: Craft targeted SQL injection payloads
    Note over A,B: Attacker uses information to refine attacks
```

## Exploit Scripts

The following exploit scripts demonstrate how to exploit each vulnerability:

- [plaintext_password_storage/exploit.py](./plaintext_password_storage/exploit.py): Extracts plaintext passwords from the database
- [sensitive_data_exposure/exploit.py](./sensitive_data_exposure/exploit.py): Extracts sensitive user data from the profile endpoint
- [sql_injection_points/exploit.py](./sql_injection_points/exploit.py): Exploits SQL injection to extract data from any table
- [debug_information_exposure/exploit.py](./debug_information_exposure/exploit.py): Gathers debug information by triggering errors
- [detailed_error_messages/exploit.py](./detailed_error_messages/exploit.py): Exploits detailed error messages to gather database information

## Remediation Recommendations

1. **Secure Password Storage**: Use bcrypt or another strong hashing algorithm with salt for password storage.
2. **Minimize Data Exposure**: Return only necessary data in API responses.
3. **Parameterized Queries**: Use parameterized queries for all database operations.
4. **Generic Error Handling**: Implement generic error messages that don't reveal sensitive information.
5. **Disable Debug Information**: Disable debug information in production environments.
