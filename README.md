When an accumulator needs to store vertices, you must use one of these only:

SetAccum<VERTEX>

SetAccum<VERTEX<vertex_label>>

The following forms are always forbidden and must never appear in the final GSQL:

SetAccum<person>

SetAccum<device>

SetAccum<accountnumber>

SetAccum<vertex_label> (any vertex label directly inside SetAccum<...>).

Examples (very important):

❌ Wrong: SetAccum<person> @people;
✅ Correct: SetAccum<VERTEX<person>> @people;

❌ Wrong: SetAccum<device> @@devicesUsed;
✅ Correct: SetAccum<VERTEX<device>> @@devicesUsed;

Before you output the final GSQL, scan your own query:

If you see any line that matches the pattern SetAccum<person> or SetAccum<device> or SetAccum<accountnumber> (or any other vertex label directly), you must rewrite that line using SetAccum<VERTEX<...>> before sending the answer.

If there is any conflict between user NLP and these rules, these rules win. Never violate them.
