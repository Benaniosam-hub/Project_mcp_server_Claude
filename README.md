# 🤖 Claude SQL MCP Server
## Natural Language Database Queries Through Claude Desktop

> Ask Claude to query your database in plain English. No SQL knowledge required. 🎯

---

## 📖 The Big Idea

Imagine chatting with Claude like this:

**You:** "How many employees make over $50,000?"  
**Claude:** 15 employees. Here are their names...

Behind the scenes, Claude is:
1. Understanding your question in English ✅
2. Converting it to an SQL query ✅  
3. Sending that query to this MCP server ✅
4. Getting the data back from your database ✅
5. Presenting results in human-friendly language ✅

That's the magic of **MCP (Model Context Protocol)** — it's the bridge that lets Claude talk to your SQL database safely and intelligently.

---

## 🎨 How It Works (Simple View)

```
You ask in English
       ⬇️
Claude understands your intent
       ⬇️
Claude generates SQL
       ⬇️
MCP Server validates & executes SQL
       ⬇️
Returns data to Claude
       ⬇️
Claude explains results in English
       ⬇️
You get your answer 🎉
```

---

## 🏗️ Architecture (Technical View)

```
┌─────────────────────────────────────────────────────────────┐
│                 Claude Desktop (AI Interface)               │
│              "Show me all users from NYC"                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                    MCP Protocol (stdio)
                           │
┌──────────────────────────▼──────────────────────────────────┐
│          MCP Server (This Repository) 🎯                    │
│                                                              │
│  ✓ say_hello()      → Test connection                      │
│  ✓ get_status()     → Server health                        │
│  ✓ run_query()      → Execute SQL safely                   │
│                                                              │
│  🔒 Safety Layer:                                           │
│     • Blocks DROP DATABASE, DROP TABLE, TRUNCATE           │
│     • Validates all queries before execution               │
│     • Only allows safe operations                          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                        SQL Query
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                  PostgreSQL Database                         │
│          (Your local or remote database)                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start (5 Minutes)

### 1️⃣ Prerequisites
```bash
✅ Python 3.13+
✅ PostgreSQL (running locally or remotely)
✅ Claude Desktop (installed)
```

### 2️⃣ Clone & Setup
```bash
git clone https://github.com/Benaniosam-hub/Project_mcp_server_Claude.git
cd Project_mcp_server_Claude

# Install dependencies
uv sync
# OR
pip install fastmcp mcp psycopg2
```

### 3️⃣ Configure Database
Edit `server.py` and update your database credentials:

```python
conn = psycopg2.connect(
    host="localhost",          # Your database host
    database="db1",            # Your database name
    user="postgres",           # Your username
    password="0000",           # Your password
    port="5432"                # PostgreSQL port
)
```

### 4️⃣ Register with Claude Desktop
Open your Claude config file:

**Mac:** `~/.claude_desktop_config.json`  
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

Add this:
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

### 5️⃣ Restart Claude & Start Querying!
Restart Claude Desktop. Now try asking:

> "Show me all employees"  
> "How many users are in the database?"  
> "Add a new record for John Doe"  
> "What's the average salary?"

Claude will handle the rest! 🎉

---

## 📚 What You Can Do

| Action | Example | Works? |
|--------|---------|--------|
| **Read Data** | "List all employees" | ✅ |
| **Count Records** | "How many users are there?" | ✅ |
| **Filter Data** | "Show me employees with salary > 50k" | ✅ |
| **Insert Data** | "Add a new user John with email john@example.com" | ✅ |
| **Update Data** | "Change John's salary to 60000" | ✅ |
| **Delete Data** | "Remove user with ID 5" | ✅ |
| **Join Tables** | "Show me employees and their departments" | ✅ |
| **Delete Tables** | "Drop the employees table" | ❌ BLOCKED |
| **Drop Database** | "Delete the entire database" | ❌ BLOCKED |
| **Truncate Tables** | "Clear all data from users" | ❌ BLOCKED |

---

## 🔧 Available Tools

### 1. `say_hello()`
Test if the server is working.
```
Returns: "Hello Benanio, MCP server is working."
```

### 2. `get_status()`
Check server health.
```
Returns: "MCP server is running successfully."
```

### 3. `run_query(query: str)` ⭐ Main Tool
Execute any SQL query safely.

**Input:** Your SQL query as a string  
**Output:** 
- For SELECT: Returns data rows
- For INSERT/UPDATE/DELETE: Returns success message
- For dangerous ops: Returns error message

---

## 🔒 Safety Features

This server is designed to prevent accidents:

```python
# Automatically blocks these operations:
❌ DROP DATABASE
❌ DROP TABLE
❌ TRUNCATE
❌ ALTER DATABASE
```

**How it works:**
1. You ask Claude something in English
2. Claude generates SQL based on your request
3. MCP Server checks the query for dangerous keywords
4. If dangerous → **BLOCKED** ⛔
5. If safe → **EXECUTED** ✅

---

## 💡 Real-World Examples

### Example 1: Reading Data
```
You: "Show me all employees earning more than $50,000"

Claude generates:
SELECT * FROM employees WHERE salary > 50000;

MCP Server executes → Returns data

Claude presents:
✓ Found 5 employees:
  - Alice Johnson: $75,000
  - Bob Smith: $60,000
  - Carol Davis: $80,000
```

### Example 2: Adding Data
```
You: "Add a new student named Sarah with email sarah@school.edu"

Claude generates:
INSERT INTO students (name, email) 
VALUES ('Sarah', 'sarah@school.edu');

MCP Server executes → Returns success

Claude confirms:
✓ Student Sarah has been added to the database!
```

### Example 3: Safety Protection
```
You: "Delete the entire employees table"

Claude generates:
DROP TABLE employees;

MCP Server checks → BLOCKED ⛔

Claude informs you:
❌ I cannot delete entire tables for safety reasons.
   Please contact an administrator for this operation.
```

---

## 📁 Project Structure

```
Project_mcp_server_Claude/
│
├── server.py              ← Main MCP server (the core!)
├── main.py                ← Entry point
├── pyproject.toml         ← Dependencies & project config
├── uv.lock                ← Locked versions
├── .python-version        ← Python version (3.13)
├── .gitignore             ← Git ignore rules
├── Output_images/         ← Screenshots & docs
└── README.md              ← This file
```

---

## ⚙️ Configuration & Customization

### Change Database Connection
Edit `server.py` (lines 19-25):
```python
conn = psycopg2.connect(
    host="your-host",
    database="your-db",
    user="your-user",
    password="your-password",
    port="5432"
)
```

### Add More Blocked Keywords
In `server.py`, expand the `blocked` list:
```python
blocked = [
    "DROP DATABASE",
    "DROP TABLE",
    "TRUNCATE",
    "ALTER DATABASE",
    "DELETE FROM",      # Add this to block deletes
    "EXEC",             # Add this to block execution
]
```

### Add Custom Tools
Create new functions and decorate them:
```python
@mcp.tool()
def get_employee_count() -> int:
    """Get total number of employees"""
    # Your code here
    return count

@mcp.tool()
def get_users_by_role(role: str) -> list:
    """Get all users with a specific role"""
    # Your code here
    return users_list
```

---

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| **"Connection refused"** | Check PostgreSQL is running. Verify host/port. |
| **"Authentication failed"** | Double-check username & password in `server.py` |
| **"Database does not exist"** | Create the database or update the connection string |
| **"Claude can't see MCP server"** | Restart Claude Desktop after editing config |
| **"Query blocked unexpectedly"** | Check if query contains blocked keywords (case-insensitive) |
| **"ModuleNotFoundError"** | Run `uv sync` or `pip install fastmcp mcp psycopg2` |

---

## 🔐 Security Best Practices

### 1. Use Environment Variables (Don't hardcode passwords!)
```python
import os

host = os.getenv("DB_HOST", "localhost")
password = os.getenv("DB_PASSWORD")
user = os.getenv("DB_USER", "postgres")
```

### 2. Create Read-Only Database User
```sql
CREATE USER readonly WITH PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
```

### 3. Enable SSL for Remote Databases
```python
conn = psycopg2.connect(
    # ... other params ...
    sslmode="require"
)
```

### 4. Use Connection Pooling
For production, use a connection pool (psycopg2 pool or pgBouncer).

### 5. Enable Query Logging
Monitor what queries are being executed on your database.

---

## 📊 What Makes This Powerful

✨ **Zero SQL Knowledge Required**
- Users can query databases without learning SQL
- Perfect for non-technical team members

✨ **AI-Powered Intelligence**
- Claude understands context and natural language
- Generates correct SQL queries automatically

✨ **Built-In Safety**
- Protects against accidental data loss
- Blocks dangerous operations automatically

✨ **Seamless Integration**
- Works right inside Claude Desktop
- No separate applications or interfaces needed

✨ **Extensible**
- Easy to add more tools
- Easy to customize behavior

---

## 🎯 Perfect For

✅ **Data Analysts** - Query databases without SQL  
✅ **Business Users** - Get insights from data easily  
✅ **Teams** - Collaborative database exploration  
✅ **Learning** - Understand SQL through Claude's explanations  
✅ **Prototyping** - Quickly build database interactions  
✅ **Automation** - Build AI-powered workflows  

---

## 📚 Learn More

- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [FastMCP](https://github.com/jlowin/fastmcp)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Claude Documentation](https://claude.ai/docs)

---

## 🚀 Future Roadmap

- [ ] Support for MySQL, SQLite, MongoDB
- [ ] Query result caching
- [ ] Automatic schema introspection
- [ ] Advanced query templates
- [ ] Query history and logging
- [ ] Rate limiting
- [ ] Export to CSV/JSON/Excel
- [ ] Audit trail for compliance

---

## 🤝 Contributing

Found a bug? Have an idea? Open an issue!

[📝 Create an Issue](https://github.com/Benaniosam-hub/Project_mcp_server_Claude/issues)

---

## 📄 License

Open Source (MIT License)

---

## 👨‍💻 Author

**Benaniosam**  
🔗 [GitHub](https://github.com/Benaniosam-hub)

---

## ✨ Key Takeaway

This project turns your database into a conversational interface. Instead of:
- Learning SQL
- Writing queries
- Parsing results

You simply **ask Claude in English** and get answers back. The MCP server handles all the technical complexity behind the scenes.

**It's that simple!** 🎉

---

**Version:** 1.0.0  
**Last Updated:** July 2026
