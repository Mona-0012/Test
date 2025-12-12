import json
import configparser
import os
import sys
from typing import TypedDict, Annotated, Literal
import operator

from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage
from langgraph.graph import StateGraph, END
import pyTigerGraph as tg

# --- 1. Configuration Loading ---

def load_config():
    """Loads configuration and schema files."""
    base_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    config_ini_path = os.path.join(base_dir, 'config', 'US', 'config.ini')
    vertices_path = os.path.join(base_dir, 'config', 'vertices.json')
    edges_path = os.path.join(base_dir, 'config', 'edges.json')

    # Load INI
    config = configparser.ConfigParser()
    if not os.path.exists(config_ini_path):
        raise FileNotFoundError(f"Config file not found: {config_ini_path}")
    config.read(config_ini_path)

    # Load JSON Schemas
    with open(vertices_path, 'r') as f:
        vertices_schema = json.load(f)
    with open(edges_path, 'r') as f:
        edges_schema = json.load(f)

    return config, vertices_schema, edges_schema

# --- 2. State Definition ---

class GraphState(TypedDict):
    input_prompt: str
    schema_context: str
    generated_query: str
    validation_error: str
    retries: int
    is_valid: bool
    execution_result: str

# --- 3. Helper Functions ---

def execute_gsql_raw(query: str, config):
    """Executes the GSQL query against the TigerGraph database and returns raw output."""
    try:
        tg_config = config['TIGERGRAPH']
        host = tg_config.get('host', 'http://127.0.0.1')
        graph = tg_config.get('graphname', 'Graph_new')
        username = tg_config.get('username', 'tigergraph')
        password = tg_config.get('password', 'tigergraph')
        
        # Clean host URL
        if "://" in host:
            protocol, address = host.split("://", 1)
            host = address # Strip protocol to re-evaluate it
        
        # Determine protocol/URL
        if any(char.isdigit() for char in host.split('.')[0]):
             host = f"http://{host}"
        else:
             host = f"https://{host}" 

        print(f"Connecting to {host}, Graph: {graph}, User: {username}")
        
        conn = tg.TigerGraphConnection(
            host=host, 
            graphname=graph, 
            username=username, 
            password=password,
            useCert=False 
        )
        
        try:
             conn.getToken(conn.createSecret())
        except Exception:
             try:
                conn.getToken(conn.createSecret())
             except:
                pass 
        
        print(f"Running query...")
        response = conn.gsql(query)
        return response

    except Exception as e:
        return f"Execution Exception: {str(e)}"

# --- 4. Nodes ---

def writer_node(state: GraphState):
    """
    Generates GSQL based on prompt and schema.
    """
    print("--- WRITER NODE ---")
    config, vertices_schema, _ = load_config()
    
    # Initialize LLM
    try:
        api_key = config['LLM']['api_key'] 
        llm = ChatOpenAI(api_key=api_key, model="gpt-4o", temperature=0)
    except KeyError:
        print("Error: [LLM] api_key not found in config.ini")
        sys.exit(1)

    prompt = state['input_prompt']
    schema = state['schema_context']
    error = state.get('validation_error', '')

    system_prompt = f"""You are an expert TigerGraph GSQL query generator for INTERPRET QUERY mode.
    
    CONTEXT (Graph Schema):
    {schema}

    ===== STRICT GSQL SYNTAX RULES =====

    1. **EDGE DIRECTION RULE (MANDATORY)**:
       - TigerGraph does NOT allow undirected hops like `-(EDGE)-`.
       - ALWAYS use directed arrow syntax: `-(EDGE:e)-> VertexType:alias`
       - If schema defines `person -> device` via HAS_USED, use: `-(HAS_USED:e)-> device:d`
       - If querying from `device -> person`, use the REVERSE edge: `-(reverse_HAS_USED:e)-> person:p`

    2. **START VERTEX RULE**:
       - The start set MUST match the FROM side of the first hop.
       - Example: `start = {{person.*}};` then `FROM start:p -(HAS_USED:e)-> device:d`
       - NEVER start from the TO vertex unless reverse edge is available.

    3. **ONE HOP PER SELECT BLOCK (MANDATORY - MOST IMPORTANT RULE)**:
       - INTERPRET QUERY mode allows ONLY ONE edge traversal per SELECT statement.
       - NEVER chain multiple hops in one SELECT block.
       
       ILLEGAL (causes parsing errors):
       ```
       result = SELECT t
                FROM start:s
                     -(EDGE1:e1)-> middle:m
                     -(EDGE2:e2)-> target:t   // <-- ILLEGAL! Second hop in same SELECT
       ```
       
       CORRECT (use separate SELECT statements for each hop):
       ```
       // Hop 1: person -> device
       devices = SELECT d
                 FROM start:p
                      -(HAS_USED:e)-> device:d
       ;
       
       // Hop 2: device -> person (separate SELECT)
       people = SELECT p2
                FROM devices:d
                     -(reverse_HAS_USED:e)-> person:p2
       ;
       ```

    4. **SELECT BLOCK RULE**:
       - Each SELECT block does ONE logical step only (ONE hop).
       - For multi-hop traversals (e.g. person->device->person), use MULTIPLE SEPARATE SELECT statements.
       - Do NOT mix multiple edge traversals in one SELECT.
       
       **COMPLETE EXAMPLE: Finding people who transferred money (person->account->account->person)**:
       ```gsql
       // Accumulator declaration at TOP
       SetAccum<STRING> @receiverIds;
       
       start = {{person.*}};
       
       // Hop 1: person -> account
       accounts = SELECT a
                  FROM start:p
                       -(HAS_ACCOUNT:e)-> accountnumber:a
       ;
       
       // Hop 2: account -> account (via HAS_PAID)
       paidAccounts = SELECT a2
                      FROM accounts:a
                           -(HAS_PAID:e)-> accountnumber:a2
       ;
       
       // Hop 3: account -> person (receiver)
       receivers = SELECT p2
                   FROM paidAccounts:a
                        -(reverse_HAS_ACCOUNT:e)-> person:p2
       ;
       
       PRINT receivers;
       ```
       
       **COUNTING PATTERN EXAMPLE: Count edges per source vertex**:
       When you need to count how many edges a vertex has (e.g., "persons with more than 1 device"):
       - IMPORTANT: SELECT the SOURCE vertex (p), NOT the target (d)
       - This ensures the accumulator is properly attached to the counted vertex
       ```gsql
       SumAccum<INT> @deviceCount;
       
       start = {{person.*}};
       
       // Count devices per person - SELECT p (source), not d (target)!
       countedPersons = SELECT p
                        FROM start:p
                             -(HAS_USED:e)-> device:d
                        ACCUM
                            p.@deviceCount += 1
       ;
       
       // Filter persons with more than one device
       result = SELECT p
                FROM countedPersons:p
                WHERE p.@deviceCount > 1
       ;
       
       PRINT result;
       ```
       
       **FINDING SOURCE PATTERN (VERY IMPORTANT)**:
       When the question asks "find X that did something to Y with condition", you want:
       - SELECT X (the source/actor), NOT Y (the target)
       - WHERE filters on Y's attributes, but you return X
       
       Example: "Find persons who paid to victim accounts"
       - X = sender accounts (the payers)
       - Y = victim accounts (the receivers)
       - You want to return the PAYERS, not the victims!
       
       ```gsql
       // Hop 2: SELECT a (sender), filter by a2.victim (receiver condition)
       senderAccounts = SELECT a
                        FROM accounts:a
                             -(HAS_PAID:e)-> accountnumber:a2
                        WHERE a2.victim == true
       ;
       
       // Hop 3: Get persons from SENDER accounts
       payers = SELECT p
                FROM senderAccounts:a
                     -(reverse_HAS_ACCOUNT:e)-> person:p
       ;
       ```
       
       WRONG: `SELECT a2` would give you the victim account owners, NOT the payers!

    5. **ACCUM BLOCK FORMATTING**:
       - ACCUM MUST be on a new line after WHERE (or after FROM if no WHERE).
       - CORRECT:
         ```
         SELECT d FROM start:p
              -(HAS_USED:e)-> device:d
         ACCUM
             d.@count += 1
         ;
         ```

    6. **ACCUMULATOR DECLARATION RULE**:
       - ALL accumulators MUST be declared at the TOP of the query body, immediately after `{{`.
       - CORRECT:
         ```
         INTERPRET QUERY () FOR GRAPH Graph_new {{
             SumAccum<INT> @personCount;
             SetAccum<STRING> @ids;

             start = {{person.*}};
             ...
         }}
         ```
       - NEVER declare accumulators inside SELECT blocks.

    7. **QUERY FORMAT**:
       - Output MUST be a complete INTERPRET QUERY block:
         ```gsql
         USE GRAPH Graph_new
         INTERPRET QUERY () FOR GRAPH Graph_new {{
             // Accumulator declarations here
             SumAccum<INT> @count;

             // Start set
             start = {{VertexType.*}};

             // SELECT block (one hop per line)
             result = SELECT t
                      FROM start:s
                           -(EdgeName:e)-> TargetType:t
                      ACCUM
                          t.@count += 1
             ;

             PRINT result;
         }}
         ```

    8. **FORBIDDEN SYNTAX (NEVER OUTPUT)**:
       - Undirected edges: `-(EDGE)-` (use `-(EDGE:e)->` instead)
       - SQL-style subqueries in ACCUM/SELECT
       - NOT EXISTS, EXISTS
       - HAVING clause (not supported in interpreted mode)
       - Multi-hop on one line
       - Invented attributes or vertices not in schema
       - Installed-query syntax (CREATE QUERY)
       - Missing semicolons

    9. **VARIABLE SCOPE RULE (CRITICAL)**:
       - Each SELECT block has its OWN scope.
       - You can ONLY reference vertex/edge variables defined in the CURRENT SELECT's FROM clause.
       - NEVER write `WHERE p2 != p` or similar - the variable `p` from a previous SELECT is NOT available!
       - If you need to exclude self-matches, do NOT use variable comparisons. Instead:
         - Just let all results through (the accumulator will handle duplicates)
         - Or use accumulator-based filtering in a later SELECT
       - CORRECT: Just select all and filter later if needed:
         ```
         sharedPeople = SELECT p2 FROM devices:d -(reverse_HAS_USED:e)-> person:p2;
         ```
       - WRONG (causes "undefined variable" error):
         ```
         sharedPeople = SELECT p2 FROM devices:d -(reverse_HAS_USED:e)-> person:p2 WHERE p2 != p;
         ```

    10. **CASE SENSITIVITY (CRITICAL - THIS CAUSES MOST ERRORS)**:
        - TigerGraph attribute names are CASE-SENSITIVE!
        - YOU MUST use the EXACT case as shown in the schema above.
        - The schema JSON shows the correct case. Look at it carefully before writing any attribute access.
        - Example: If schema shows `"total_sent_value"`, write `a.total_sent_value`
        - WRONG: `a.TOTAL_SENT_VALUE`, `a.Total_Sent_Value` - these will FAIL!
        - Check every attribute name against the schema before using it.

    11. **DML RESTRICTION**:
        - Generate ONLY SELECT queries.
        - If user asks for INSERT/UPDATE/DELETE, refuse politely.

    11. **OUTPUT FORMAT**:
        - Output ONLY the GSQL code inside a markdown code block: ```gsql ... ```

    12. **ERROR FIXING**:
        If previous attempt failed, the error is: {error}
        You MUST fix this error in your new query.
    """

    messages = [
        SystemMessage(content=system_prompt),
        HumanMessage(content=prompt)
    ]

    response = llm.invoke(messages)
    content = response.content
    
    # Simple extraction of code block if present
    if "```gsql" in content:
        query = content.split("```gsql")[1].split("```")[0].strip()
    elif "```" in content:
         query = content.split("```")[1].split("```")[0].strip()
    else:
        query = content.strip()

    return {"generated_query": query, "retries": state.get('retries', 0) + 1}

def execution_node(state: GraphState):
    """
    Validates safely AND Executes the GSQL.
    If execution fails (syntax error), reports it as a validation error.
    """
    print("--- EXECUTION NODE ---")
    query = state['generated_query']
    print(f"Generated Query:\n{query}\n")
    query_upper = query.upper()
    
    # 1. Safety Check
    forbidden_keywords = ["INSERT ", "UPDATE ", "DELETE ", "DROP ", "CREATE "]
    for word in forbidden_keywords:
        if word in query_upper:
            error_msg = f"Security Violation: Found forbidden keyword '{word.strip()}'. Only SELECT queries are allowed."
            print(f"Safety Failed: {error_msg}")
            return {"is_valid": False, "validation_error": error_msg, "execution_result": ""}

    if "SELECT" not in query_upper:
         if "REFUSE" in query_upper or "SORRY" in query_upper:
             return {"is_valid": True, "validation_error": "", "execution_result": "Refused"}
         
         error_msg = "Syntax Error: Query does not appear to be a SELECT statement."
         print(f"Safety Failed: {error_msg}")
         return {"is_valid": False, "validation_error": error_msg, "execution_result": ""}

    # Check for HAVING (not supported in interpreted mode)
    if "HAVING" in query_upper:
        error_msg = "SYNTAX ERROR: HAVING clause is NOT supported in INTERPRET QUERY mode. Use a separate SELECT with WHERE to filter instead."
        print(f"Validation Failed: {error_msg}")
        return {"is_valid": False, "validation_error": error_msg, "execution_result": ""}

    # 2. Multi-hop Chain Detection (CRITICAL - catches illegal multi-hop before DB execution)
    import re
    # Split query into SELECT blocks
    select_blocks = re.split(r'(?=\bSELECT\b)', query, flags=re.IGNORECASE)
    all_defined_aliases = set()
    for block in select_blocks:
        if not block.strip():
            continue
        # Count edge traversals (patterns like "-(EdgeName:e)-> VertexType:alias" )
        hop_count = len(re.findall(r'-\([^)]+\)->', block))
        if hop_count > 1:
            error_msg = f"MULTI-HOP ERROR: Found {hop_count} edge traversals in a single SELECT block. INTERPRET QUERY allows only ONE hop per SELECT. Use separate SELECT statements for each hop."
            print(f"Validation Failed: {error_msg}")
            return {"is_valid": False, "validation_error": error_msg, "execution_result": ""}
        
        # Extract aliases defined in this SELECT's FROM clause
        # Pattern: FROM <set>:<alias> or VertexType:<alias>
        from_aliases = set(re.findall(r'(?:FROM\s+\w+:|[a-zA-Z_]+:)(\w+)', block, re.IGNORECASE))
        
        # Check WHERE clause for undefined variables
        where_match = re.search(r'WHERE\s+(.+?)(?:ACCUM|;|\n\s*\n)', block, re.IGNORECASE | re.DOTALL)
        if where_match:
            where_clause = where_match.group(1)
            # Look for variable references like "!= p" or "== p2" or just standalone variables
            refs = re.findall(r'!=\s*(\w+)|==\s*(\w+)', where_clause)
            for ref_tuple in refs:
                for ref in ref_tuple:
                    # Only check aliases from CURRENT SELECT, NOT from previous ones
                    if ref and ref not in from_aliases:
                        # Check if it looks like a vertex alias (not a constant or keyword)
                        if re.match(r'^[a-z][a-z0-9_]*$', ref) and ref not in ['true', 'false', 'null']:
                            error_msg = f"SCOPE ERROR: Variable '{ref}' used in WHERE clause but not defined in current SELECT. Each SELECT has its own scope - you cannot reference variables from previous SELECTs. Remove comparisons like 'p2 != p'."
                            print(f"Validation Failed: {error_msg}")
                            return {"is_valid": False, "validation_error": error_msg, "execution_result": ""}

    # 3. Execution Check
    config, _, _ = load_config()
    
    # Check if we should even execute (User might not have access, but for this task we assume yes)
    print("Safety passed. Executing against DB...")
    
    result = execute_gsql_raw(query, config)
    
    # 3. Analyze Result for Errors
    # TigerGraph GSQL shell usage usually returns "Semantic Check Fails", "Syntax Error", "Exception" or similar on failure.
    # Success usually contains "JSON" or "GSQL >" prompt without error keywords usually.
    # But since we use .gsql(), it returns the stdout string.
    
    # Heuristics for failure in GSQL output
    result_lower = result.lower()
    error_keywords = [
        "syntax error", 
        "semantic check fails", 
        "semantic check error",
        "type check error",
        "undefined",
        "exception", 
        "error:", 
        "not found",
        "failed to define"
    ]
    
    if any(keyword in result_lower for keyword in error_keywords):
        print(f"DB Execution Failed with Error.")
        return {"is_valid": False, "validation_error": f"DB Execution Error:\n{result}", "execution_result": result}
    
    print("Execution Success.")
    return {"is_valid": True, "validation_error": "", "execution_result": result}

# --- 5. Conditional Logic ---

def router_node(state: GraphState) -> Literal["end", "retry"]:
    if state['is_valid']:
        return "end"
    
    if state['retries'] >= 5:
        print("Max retries reached.")
        return "end"
    
    return "retry"

# --- 6. Graph Construction ---

def build_graph():
    workflow = StateGraph(GraphState)

    workflow.add_node("writer", writer_node)
    workflow.add_node("executor", execution_node)

    workflow.set_entry_point("writer")
    
    workflow.add_edge("writer", "executor")
    
    workflow.add_conditional_edges(
        "executor",
        router_node,
        {
            "end": END,
            "retry": "writer"
        }
    )

    return workflow.compile()

# --- 7. Main Execution ---

if __name__ == "__main__":
    
    # --- USER CONFIGURATION ---
    PROMPT = "Show me all persons above 60 who have used a device ."

    # --------------------------

    if len(sys.argv) > 1:
        PROMPT = sys.argv[1]

    print(f"Using PROMPT: {PROMPT}")
    
    try:
        config, vertices, edges = load_config()
        schema_context = json.dumps({"vertices": vertices, "edges": edges}, indent=2)
    except Exception as e:
        print(f"Initialization Error: {e}")
        sys.exit(1)

    app = build_graph()
    
    initial_state = {
        "input_prompt": PROMPT,
        "schema_context": schema_context,
        "retries": 0,
        "validation_error": "",
        "is_valid": False, 
        "generated_query": "",
        "execution_result": ""
    }

    try:
        final_state = app.invoke(initial_state)
        
        print("\n=== FINAL OUTPUT ===")
        if final_state['is_valid']:
            print("Generated GSQL:")
            print(final_state['generated_query'])
            print("\nExecution Result:")
            print(final_state['execution_result'])
        else:
            print("Failed to generate valid query specific constraints.")
            print(f"Last Error: {final_state.get('validation_error')}")
            
    except Exception as e:
        print(f"Execution Error: {e}")
