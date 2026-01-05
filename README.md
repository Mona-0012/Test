When a vertex is selected from the dropdown (e.g., accountnumber), read the backend response from /tg/get-config-tables, find the matching vertex object, and dynamically populate the Vertex Metadata table.

Show each attribute as a separate row where:

Key = attribute name

Value = attribute type

Update the table reactively on vertex change and keep the existing UI layout and Save/Edit actions unchanged.

Make Vertex Metadata fully dynamic.
Whenever the selected vertex changes, dynamically load and display only the metadata of the currently selected vertex (matching vertex_name).

Do not retain or reuse data from the previously selected vertex. The table must always reflect the current dropdown selection.
