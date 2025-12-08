When generating GSQL, never use accumulator method calls like .size(), .count(), or any accumulator function inside WHERE. This is invalid GSQL.

Instead, always compute such conditions in POST-ACCUM using a BoolAccum flag, and filter using that flag in the WHERE clause.

Also, never generate start sets like {start_devices.*}. Always assign start = {<vertexType>.*} directly, and never reference a variable before declaring it.

Apply these two rules and regenerate the corrected GSQL for the given NLP.
