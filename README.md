I am implementing a retry mechanism for my text-to-GSQL pipeline.

I need you to inspect the code in this file and verify the following:
1. After generating the first GSQL and executing it on TigerGraph, does the code correctly detect the TG error?
2. Does the code actually call my retry prompt builder (get_retry_prompt or build_retry_task_text) when an error happens?
3. Does the retry logic correctly pass these four inputs into the retry task:
   - original NLP text
   - invalid GSQL from the first LLM call
   - TigerGraph error message
   - the schema/nodes/edges data
4. Does the second LLM call (retry) actually run and produce new GSQL?
5. After retry, is the code executing the new GSQL on TigerGraph?

If any of these steps are missing, incorrect, or wired in the wrong order, show me exactly where and what I need to fix in the code. 
Do not rewrite the entire file—only point out the specific missing calls or wrong wiring.
