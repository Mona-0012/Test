When a vertex is selected from the dropdown (e.g., accountnumber), read the backend response from /tg/get-config-tables, find the matching vertex object, and dynamically populate the Vertex Metadata table.

Show each attribute as a separate row where:

Key = attribute name

Value = attribute type

Update the table reactively on vertex change and keep the existing UI layout and Save/Edit actions unchanged.
