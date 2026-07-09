# MCP Server for Claude × SQL Database

A **Model Context Protocol (MCP) server** that bridges Claude AI with SQL databases, enabling natural language queries and database operations through Claude Desktop.

## 🎯 Overview

This project implements an **MCP server** that acts as a middleware between Claude and PostgreSQL databases. Instead of writing SQL queries manually, users can ask Claude questions in plain English, and the MCP server:

1. **Receives natural language requests** from Claude
2. **Translates them to SQL queries** (Claude handles this)
3. **Executes queries safely** against PostgreSQL
4. **Returns results** back to Claude in human-readable format

The result? A seamless conversational interface for database interactions with built-in safety guards against destructive operations.

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    Claude Desktop (AI)                           │
│                   (User asks questions)                          │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                    Human Language Query
                             │
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│           MCP Server (Model Context Protocol)                     │
│                   (This Repository)                              │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Tools Exposed:                                         │    │
│  │  • say_hello() - Test connection                        │    │
│  │  • get_status() - Server status                         │    │
│  │  • run_query(query: str) - Execute SQL queries          │    │
│  │                                                          │    │
│  │  Safety Features:                                       │    │
│  │  ✓ Blocks DROP DATABASE, DROP TABLE, TRUNCATE, etc.    │    │
│  │  ✓ Distinguishes SELECT (read) vs write operations     │    │
│  │  ✓ Automatic connection management                     │    │
│  └─────────────────────────────────────────────────────────┘    │
└────────────────────────────┬─────────────────────────────────────┘
                             │
                    SQL Query (safe & validated)
                             │
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│              PostgreSQL Database                                 │
│         (localhost:5432 or any remote instance)                  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Database: db1                                           │   │
│  │  Tables: employees, students, users, etc.               │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.13+
- PostgreSQL (local or remote)
- Claude Desktop
- `uv` (Python package installer) — or `pip`

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Benaniosam-hub/Project_mcp_server_Claude.git
   cd Project_mcp_server_Claude
   ```

2. **Install dependencies:**
   ```bash
   # Using uv (recommended)
   uv sync
   
   # Or using pip
   pip install -r requirements.txt
   ```

3. **Configure database connection in `server.py`:**
   
   Update the connection parameters in the `run_query()` function:
   ```python
   conn = psycopg2.connect(
       host="localhost",        # Your DB host
       database="db1",          # Your database name
       user="postgres",         # Your DB user
       password="0000",         # Your DB password
       port="5432"              # PostgreSQL port
   )
   ```

4. **Run the MCP server:**
   ```bash
   python server.py
   ```
   
   You should see: `MCP server is running successfully.`

---

## 🔌 Integration with Claude Desktop

### Step 1: Configure Claude Desktop

On Mac, edit `~/.claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "mcp-sql-server": {
      "command": "python",
      "args": ["/path/to/Project_mcp_server_Claude/server.py"]
    }
  }
}
```

On Windows, edit `%APPDATA%\Claude\claude_desktop_config.json` with the equivalent path.

### Step 2: Restart Claude Desktop

Restart Claude Desktop, and the MCP server tools will be available.

### Step 3: Start Querying

Now in Claude, you can ask:

> "Show me all employees in the database"
> "How many students are enrolled?"
> "Get the latest 10 records from the users table"
> "Insert a new employee record with name 'John' and salary 50000"

Claude will:
1. Understand your intent
2. Translate it to SQL
3. Execute via the MCP server
4. Present results in natural language

---

## 📋 Available Tools

### 1. `say_hello()`
- **Purpose:** Test if the MCP server is responding
- **Returns:** `"Hello Benanio, MCP server is working."`

### 2. `get_status()`
- **Purpose:** Check server status
- **Returns:** `"MCP server is running successfully."`

### 3. `run_query(query: str)`
- **Purpose:** Execute SQL queries against PostgreSQL
- **Input:** SQL query string (SELECT, INSERT, UPDATE, DELETE)
- **Returns:** 
  - For SELECT: Rows of data
  - For INSERT/UPDATE/DELETE: Success message
  - For blocked operations: Error message

---

## 🔒 Safety Features

The MCP server includes built-in protections:

| Blocked Operations | Reason |
|---|---|
| `DROP DATABASE` | Prevents database deletion |
| `DROP TABLE` | Prevents table deletion |
| `TRUNCATE` | Prevents data wipeout |
| `ALTER DATABASE` | Prevents structural changes |

**How it works:**
- Before executing any query, the server checks if it contains blocked keywords
- If detected, the query is rejected with a clear message
- Only read-safe (`SELECT`) and approved write operations are allowed

---

## 📁 Project Structure

```
Project_mcp_server_Claude/
├── server.py              # Main MCP server implementation
├── main.py                # Entry point placeholder
├── pyproject.toml         # Python project metadata & dependencies
├── uv.lock                # Locked dependency versions
├── .gitignore             # Git ignore rules
├── .python-version        # Python version spec
├── Output_images/         # Screenshots & documentation
└── README.md              # This file
```

---

## 🛠️ How It Works Under the Hood

1. **MCP Protocol:** Uses FastMCP framework to expose tools via the Model Context Protocol
2. **Tool Registration:** Each function decorated with `@mcp.tool()` becomes a callable tool in Claude
3. **Query Execution:** 
   - Claude decides to use `run_query()` based on user intent
   - Passes the SQL query string
   - Server validates and executes against PostgreSQL
4. **Result Handling:**
   - SELECT queries: Returns data rows
   - Write operations: Returns confirmation message
   - Safety checks: Blocks dangerous operations

---

## 📝 Example Conversations

### Example 1: Reading Data
**User:** "List all employees with salary > 50000"

Claude translates to SQL:
```sql
SELECT * FROM employees WHERE salary > 50000;
```

Server executes and Claude presents:
```
Found 5 employees:
- Alice: $75,000
- Bob: $60,000
- Carol: $80,000
- Dave: $65,000
- Eve: $55,000
```

### Example 2: Inserting Data
**User:** "Add a new student named Alex with email alex@example.com"

Claude translates to SQL:
```sql
INSERT INTO students (name, email) VALUES ('Alex', 'alex@example.com');
```

Server returns: `Query executed successfully.`

Claude confirms: `✓ Student 'Alex' has been added to the database.`

### Example 3: Blocked Operation
**User:** "Delete the entire employees table"

Claude might attempt:
```sql
DROP TABLE employees;
```

Server blocks it: `DROP TABLE is blocked.`

Claude informs user: `❌ I cannot delete entire tables for safety reasons. Please contact an administrator.`

---

## ⚙️ Configuration

### Database Connection
Edit `server.py` lines 19-25 to match your PostgreSQL setup:

```python
conn = psycopg2.connect(
    host="localhost",       # PostgreSQL host
    database="db1",         # Database name
    user="postgres",        # Username
    password="0000",        # Password
    port="5432"             # Port
)
```

### Adding New Safety Rules
To add more blocked keywords, update the `blocked` list in `run_query()`:

```python
blocked = [
    "DROP DATABASE",
    "DROP TABLE",
    "TRUNCATE",
    "ALTER DATABASE",
    # Add more here:
    # "DELETE FROM",  # To prevent deletes
]
```

### Expanding Tool Set
Add new tools by creating functions and decorating them:

```python
@mcp.tool()
def get_employee_count() -> int:
    # Implementation here
    return count
```

---

## 🐛 Troubleshooting

| Issue | Solution |
|---|---|
| `Connection refused` | Check PostgreSQL is running and host/port are correct |
| `Authentication failed` | Verify database username and password |
| `Database db1 does not exist` | Create the database or update the connection string |
| `Claude can't see the MCP server` | Restart Claude Desktop after config changes |
| `Query is blocked unexpectedly` | Check if query contains blocked keywords (case-insensitive) |

---

## 🔐 Security Best Practices

1. **Never commit credentials** - Use environment variables instead:
   ```python
   import os
   host = os.getenv("DB_HOST", "localhost")
   password = os.getenv("DB_PASSWORD")
   ```

2. **Restrict database user permissions** - Create a read-only user for queries:
   ```sql
   CREATE USER readonly WITH PASSWORD 'password';
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
   ```

3. **Use connection pooling** - For production, use connection pools instead of creating connections per query

4. **Enable SSL** - For remote databases, enable SSL/TLS connections

5. **Audit queries** - Enable PostgreSQL query logging to track operations

---

## 📚 Dependencies

- **fastmcp** (≥3.4.2): Fast implementation of Model Context Protocol
- **mcp** (≥1.27.2): Model Context Protocol SDK
- **psycopg2**: PostgreSQL database adapter for Python

---

## 🎓 Learning Resources

- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [FastMCP GitHub](https://github.com/jlowin/fastmcp)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Claude AI Documentation](https://claude.ai/docs)

---

## 🚀 Future Enhancements

- [ ] Support for multiple databases (MySQL, SQLite, etc.)
- [ ] Query result caching for performance
- [ ] Custom query templates
- [ ] Automatic schema introspection to help Claude understand table structure
- [ ] Query logging and analytics
- [ ] Rate limiting and query quotas
- [ ] Advanced filtering UI
- [ ] Export results to CSV/JSON

---

## 📄 License

Open source (specify LICENSE file when ready - MIT, Apache 2.0, etc.)

---

## 👨‍💻 Author

**Benaniosam** — https://github.com/Benaniosam-hub

---

## 💬 Support

Found a bug or have a suggestion? Open an issue:
https://github.com/Benaniosam-hub/Project_mcp_server_Claude/issues

---

## 🎉 Highlights

✨ **What Makes This Special:**
- Natural language interface to databases (no SQL knowledge needed)
- AI-powered query generation via Claude
- Built-in safety mechanisms prevent accidental data loss
- Seamless Claude Desktop integration
- Production-ready code structure

**Perfect for:**
- Data exploration without writing SQL
- Prototyping database-backed applications
- Learning SQL through AI assistance
- Team members without SQL expertise
- Quick database queries during analysis tasks

---

**Last Updated:** July 2026  
**Version:** 1.0.0
