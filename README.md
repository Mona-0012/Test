Add a new dropdown above the existing "Select Table" dropdown in this component.

Requirements:

1. Create a new state variable:
      const [selectedSource, setSelectedSource] = useState("BQ");

2. Render a dropdown labeled "Choose Source" with two options:
      - BigQuery (value: "BQ")
      - TigerGraph (value: "TG")

3. When the user selects a value, call setSelectedSource(option.value).

4. Pass this selectedSource to the existing logic as props.selectedSource.  
   (If needed, update props or lift state up so useEffect receives it.)

5. Do NOT modify the layout of the existing UI.  
   Only insert this one dropdown above the "Select Table" dropdown.

6. Do NOT change the existing Select Table component or BQ logic.

7. The existing useEffect will switch automatically based on selectedSource,  
   so ensure the new dropdown updates selectedSource correctly.

Generate only the JSX code snippet for this new dropdown and the state variable.
