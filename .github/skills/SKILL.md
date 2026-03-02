
---
name: impc_api_query
description: A skill to use when a user asks to query a gene, disease and phenotype data from the IMPC API using the `impc_api` python package. 
--- 

# IMPC API QUERY


> EXECUTION ENVIRONMENT — READ FIRST
> All code MUST be executed via the `pylance` MCP server tool.
> Do NOT use any other execution method under any circumstances.
> A uv environment at `/path/to/env` has all dependencies pre-installed.

## Context
The IMPC Solr API should be queried safely with the `impc_api` python package. It contains cores and fields to query. Use the pylance mcp server to execute python code. 

## Workflow
1. Find the existing cores and available fields per core:
    ``` uv run 
    import httpx
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get("https://raw.githubusercontent.com/mpi2/impc-api/refs/heads/main/impc_api/utils/core_fields.json")
            response.raise_for_status()
            return response.json()
    except Exception:
        return None
    ```

2. Use the available cores and fields to craft a query as follows where both core and params are needed.

REQUIRED ARGUMENTS:
    core (str): The Solr core to query.
    params (dict): Query parameters. Must include at minimum:
        - "q": query string. Use "*:*" for all records, or "field:value" 
               for specific matches. Example: "marker_symbol:Pparg"
        - "rows": number of results to return (int). Default to 10 unless 
                  the user specifies otherwise.
        - "fl": comma-separated string of fields to return. 
                Example: "marker_symbol,marker_name,marker_description"

CORE-SPECIFIC RULES:

    phenodigm core:
        1. ALWAYS include a type filter in "q". Never query phenodigm without a type.
           Valid types and when to use them:
               - type:disease_model_summary  → gene-disease associations
               - type:disease                → disease records
               - type:gene                  → gene records
               - type:gene_gene             → human-mouse gene mapping
               - type:ontology_ontology     → HPO-MP ontology mapping
               - type:ontology              → ontology records
               - type:mouse_model           → mouse model recrods
               - 
           Example: {"q": "type:disease_model_summary AND marker_symbol:Pparg"}

        2. PHENODIGM SCORE: Never store or return raw score fields directly.
           Always calculate the phenodigm score using and include it as a field:
               phenodigm_score:div(sum(disease_model_avg_norm,disease_model_max_norm),2)
           This expression can be used as a field alias or in a "sort" param.
           Example sort: "sort": "div(sum(disease_model_avg_norm,disease_model_max_norm),2) desc"

        3. INTERPRETATION: When asked about interpretation, include following fields in the query: disease_matched_phenotypes and model_matched_phenotypes.

EXAMPLE CALLS:

    Gene lookup:
        core="gene"
        params={"q": "marker_symbol:Pparg", "rows": 10, "fl": "marker_symbol,marker_name"}

    Disease-gene associations with score:
        core="phenodigm"
        params={
            "q": "type:disease_model_summary AND marker_symbol:Pparg,
            "rows": 10,
            "fl": "marker_symbol,disease_term,disease_model_avg_norm,disease_model_max_norm",
            "sort": "div(sum(disease_model_avg_norm,disease_model_max_norm),2) desc"
        }


``` uv run
    from impc_api import solr_request
    _, df = solr_request(core, params, silent=True)
    return df.to_dict(orient="records")
```

3. If asked to interpret the score follow [INTERPRET.MD](./INTERPRET.md)

    
           
