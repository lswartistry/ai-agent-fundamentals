# 3. Tools & Functions - Giving Agents Superpowers

## 🦸 Why Tools Matter

Without tools: Agent can only talk
With tools: Agent can take action

```
Without tools:
You: "What's in my repo?"
Agent: "I don't know, I can't read files" ❌

With tools:
You: "What's in my repo?"
Agent: *reads files* → "Here's what I found" ✅
```

## 🔧 What Are Tools?

Tools are **functions the agent can call** to:
- Access information (read files, search web, query databases)
- Perform actions (write code, run tests, deploy)
- Interact with external systems (call APIs, send emails)

## 📋 Common Types of Tools

### 1. **Information Tools** (Read/Gather Data)

```python
tools = {
    "read_file": lambda path: open(path).read(),
    "search_docs": lambda query: search(documentation),
    "get_user_data": lambda user_id: database.fetch(user_id),
    "web_search": lambda query: search_internet(query),
}
```

**When agent uses it:**
```
Agent thinks: "I need to know what's in this file"
          ↓
Calls: read_file("main.py")
          ↓
Gets: File contents
          ↓
Continues thinking with new info
```

### 2. **Action Tools** (Do Something)

```python
tools = {
    "write_file": lambda path, content: write(path, content),
    "run_tests": lambda: execute_tests(),
    "create_pull_request": lambda title, body: create_pr(),
    "send_email": lambda to, subject, body: send_mail(),
}
```

### 3. **Analysis Tools** (Process Data)

```python
tools = {
    "analyze_code": lambda code: static_analysis(code),
    "check_performance": lambda code: benchmark(code),
    "validate_json": lambda data: validate(data),
    "diff_files": lambda file1, file2: compare(file1, file2),
}
```

## 🎯 Real-World Example: GitHub Code Review Agent

Let's trace through what tools a code review agent needs:

```
User: "Review my pull request for security issues"

Agent's Tools:
├─ read_files()           # Read the changed files
├─ analyze_security()     # Check for security patterns
├─ search_docs()          # Look up best practices
├─ run_tests()            # Verify tests pass
├─ get_pr_context()       # Understand what changed
├─ format_comment()       # Create review comment
└─ post_comment()         # Post the review

Agent's Loop:
1. read_files() → Get changed code
2. analyze_security() → Find vulnerabilities
3. search_docs() → Find standards for this language
4. run_tests() → Verify nothing broke
5. format_comment() → Organize findings
6. post_comment() → Leave review on PR
```

## 🏗️ How Tools Are Structured

Each tool needs:

```python
{
    "name": "read_file",                    # Name agent uses
    "description": "Read contents of a file",  # What it does
    "parameters": {                         # What it needs
        "file_path": {
            "type": "string",
            "description": "Path to file to read"
        }
    },
    "function": read_file_impl              # Code that runs
}
```

**Why this structure?**
- Agent reads the description to understand what tool does
- Agent sees parameters to know what to give it
- Function actually executes

## 💡 Tool Design Principles

### 1. **Be Specific**

❌ Bad:
```python
def do_stuff():
    # Agent doesn't know what this does
```

✅ Good:
```python
def read_python_file(file_path: str) -> str:
    """
    Read a Python file and return its contents.
    file_path: Full path to the Python file
    Returns: The file contents as a string
    """
```

### 2. **Clear Inputs & Outputs**

❌ Bad:
```python
def process(x):
    # What is x? What does it return?
```

✅ Good:
```python
def validate_email(email: str) -> bool:
    """
    Check if an email is valid.
    Returns: True if valid, False otherwise
    """
```

### 3. **One Job Per Tool**

❌ Bad:
```python
def do_everything():
    # Read files
    # Analyze code
    # Write reports
    # ...
```

✅ Good:
```python
def read_file(path: str) -> str:
    """Read a file"""

def analyze_code(code: str) -> dict:
    """Analyze code"""

def write_report(analysis: dict) -> str:
    """Write a report"""
```

## 🔗 Chaining Tools Together

Agents often use multiple tools in sequence:

```
Task: "Find all security issues in my code and create a PR with fixes"

Agent Flow:
1. read_files()
   ↓ (gets code)
2. analyze_security()
   ↓ (finds issues)
3. search_best_practices()
   ↓ (learns how to fix)
4. generate_fixes()
   ↓ (creates fixed code)
5. write_files()
   ��� (saves changes)
6. create_pull_request()
   ↓ (creates PR)
7. post_comment()
   ↓ (explains changes)
```

Each tool's output becomes the next tool's input!

## 🎓 Tool Considerations

### Error Handling
Tools can fail - agent needs to handle it:

```python
tool_result = read_file("path.py")

if tool_result.error:
    # Agent should handle the error
    agent.think("File not found, ask user for correct path")
else:
    agent.use(tool_result)
```

### Permissions
Tools should check if agent is allowed:

```python
def delete_files(paths: list):
    # Security check - is agent allowed to delete?
    if not agent.has_permission("delete_files"):
        return error("Permission denied")
    delete(paths)
```

### Performance
Some tools take time:

```python
# Fast tools (< 1 second)
read_file()
search_docs()

# Slow tools (may take minutes)
run_tests()
build_application()
deploy()

Agent should plan accordingly and show progress!
```

## 📊 Tools Available to GitHub Copilot

When you interact with Copilot, it has access to tools like:

```
Code Tools:
  ├─ read_file()          # Read your code files
  ├─ search_code()        # Search your repository
  ├─ analyze_code()       # Static code analysis
  ├─ write_file()         # Create/modify files
  ├─ run_tests()          # Execute tests
  └─ check_syntax()       # Validate code

GitHub Tools:
  ├─ read_pull_request()  # Get PR details
  ├─ get_issues()         # List repository issues
  ├─ post_comment()       # Comment on PR/issue
  ├─ create_branch()      # Create a new branch
  └─ push_code()          # Push changes

Data Tools:
  ├─ search_documentation()  # Find docs
  ├─ query_database()        # Get data from DB
  └─ call_api()              # Call external APIs
```

## 🚀 Building Your Own Tool

Simple example:

```python
def count_lines_of_code(file_path: str) -> int:
    """
    Count the number of lines of code in a file.
    
    Parameters:
      file_path: Full path to the file
      
    Returns:
      Number of lines (excluding empty lines and comments)
    """
    with open(file_path, 'r') as f:
        lines = f.readlines()
        
    code_lines = 0
    for line in lines:
        stripped = line.strip()
        if stripped and not stripped.startswith('#'):
            code_lines += 1
            
    return code_lines
```

Tell agent about it:

```python
tool = {
    "name": "count_lines",
    "description": "Count lines of actual code in a file",
    "parameters": {
        "file_path": {
            "type": "string",
            "description": "Path to file to analyze"
        }
    },
    "function": count_lines_of_code
}

agent.add_tool(tool)
```

Now agent can use it:
```
You: "How many lines of code is in my main file?"
Agent: *uses count_lines tool* → "Your main.py has 234 lines of code"
```

## ✅ Tool Checklist

Before giving a tool to an agent, verify:
- ✅ Clear name: What does it do?
- ✅ Clear description: Why would agent use it?
- ✅ Clear parameters: What inputs does it need?
- ✅ Clear output: What does it return?
- ✅ Error handling: What if it fails?
- ✅ Permissions: Is it safe?
- ✅ Performance: Does it run fast enough?

---

**Next:** Learn how to communicate with agents effectively → [4. Prompting Guide](./4-prompting-guide.md)

## ❓ Common Questions

**Q: How many tools should an agent have?**
A: Start with 3-5. More tools = harder to choose. Too many confuses agents.

**Q: Can tools call other tools?**
A: Yes! Advanced agents compose tools together. Tool A calls Tool B internally.

**Q: What if a tool fails?**
A: Good agents recover. They learn from failures and try different approaches.

**Q: Do I need to write all my own tools?**
A: No! Many platforms provide pre-built tools. Copilot has built-in tools for code/GitHub.
