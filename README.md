Fix the vertex change handling so that Vertex Metadata updates correctly on every dropdown change.

On selectedVertex change:

Clear previous vertex metadata state

Re-find the vertex object from the latest /tg/get-config-tables response using vertex_name === selectedVertex

Rebuild the metadata rows from that vertex only

Do not reuse cached data from the previously selected vertex. Ensure switching from accountnumber → device shows only device attributes.
