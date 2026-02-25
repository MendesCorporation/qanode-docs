# Credentials

**Credentials** store connection information in a secure and centralized way. Instead of placing passwords and tokens directly in nodes, you create a credential and reference it in flows.

---

## Managing Credentials

1. In the side menu, click **Credentials**
2. The list displays all registered credentials

[Credentials list](../../assets/images/lista-credencial.mp4)
*Image: Credentials screen with a list showing name, type, and creation date*

---

## Creating a Credential

1. Click **+ New Credential**
2. Select the credential **type**
3. Fill in the type-specific fields
4. (Optional) Click **Test Connection** to verify
5. Click **Save**

---

## Credential Types

### HTTP / API

For connections with REST APIs.

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **Base URL** | API base URL (e.g., `https://api.exemplo.com`) |
| **Auth Type** | `None`, `Bearer Token`, `Basic Auth` |
| **Token** | Bearer token (when Bearer) |
| **Username** | Username (when Basic) |
| **Password** | Password (when Basic) |
| **Headers** | Custom headers (key-value pairs) |

**Usage in the HTTP Request node:** Select the credential and provide only the path:
```
Credential: "API Exemplo"
URL: /api/users  (the base URL is added automatically)
```

---

### PostgreSQL

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **Host** | Server address |
| **Port** | Port (default: 5432) |
| **Database** | Database name |
| **Username** | Connection username |
| **Password** | Connection password |
| **SSL** | Use SSL connection |

---

### MySQL

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **Host** | Server address |
| **Port** | Port (default: 3306) |
| **Database** | Database name |
| **Username** | Connection username |
| **Password** | Connection password |
| **SSL** | Use SSL connection |

---

### MariaDB

Same fields as MySQL.

---

### Oracle

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **Host** | Server address |
| **Port** | Port (default: 1521) |
| **Service Name** | Oracle service name |
| **Username** | Connection username |
| **Password** | Connection password |

---

### MongoDB

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **URI** | MongoDB connection string |
| **Database** | Database name (when not included in the URI) |

**Example URIs:**
```
mongodb://usuario:senha@host:27017/banco?authSource=admin
mongodb+srv://usuario:senha@cluster.mongodb.net/banco
```

---

### SSH

| Field | Description |
|-------|-------------|
| **Name** | Credential name |
| **Host** | Server address |
| **Port** | SSH port (default: 22) |
| **Username** | Username |
| **Password** | Password (password authentication) |
| **Private Key** | Private key content (key-based authentication) |
| **Passphrase** | Key passphrase (if key is protected) |

---

## Testing the Connection

Before using a credential, test the connection:

1. On the credential edit screen, click **Test Connection**
2. QANode will attempt to connect using the provided information
3. The result will be displayed: success or error with a detailed message

[Connection test](../../assets/images/teste-credencial.mp4)
*Image: Credential with a test connection button and success message*

---

## Using Credentials in Flows

### In Database Nodes

In the PostgreSQL, MySQL, MariaDB, Oracle, or MongoDB node:

1. In the **Credential** field, select the credential
2. The connection fields are filled in automatically
3. Configure only the query/operation

### In HTTP Request Nodes

1. In the **Credential** field, select the HTTP credential
2. The base URL and authentication are applied automatically
3. In the **URL** field, provide only the path

### In SSH Nodes

1. In the **Credential** field, select the SSH credential
2. The connection details are applied automatically
3. Configure only the commands

---

## Security

- Passwords and tokens are **encrypted** in the database
- Sensitive values are **masked** in the interface
- Credentials are accessible only to users with the appropriate permission
- The credential owner can edit it; others require admin permission

---

## Tips

- **Create credentials per environment** — Production, Staging, Dev
- **Always test** the connection before using it in a flow
- **Use descriptive names**: "PostgreSQL Production" is better than "pg1"
- **Prefer credentials** over placing passwords directly in nodes
- When **duplicating environments**, create new credentials for each one
