
---
name: impc-api-query
description: Queries the IMPC API using the `impc_api` python package. Use when a user asks to query a gene, disease, phenotype, allele. 
--- 

# IMPC API QUERY


> EXECUTION ENVIRONMENT — READ FIRST
> All code MUST be executed via the `pylance` MCP server tool.
> Do NOT use any other execution method under any circumstances.
> A uv environment at `/path/to/env` has all dependencies pre-installed.

## Context
The IMPC Solr API should be queried only with `impc_api` python package. It contains cores and fields to query.

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
    To run the `impc-api` request package, you need to figure out:
    core (str): The Solr core to query.
    params (dict): Query parameters. Must include at minimum:
        - "q": query string. Use "*:*" for all records, or "field:value" 
               for specific matches. Example: "marker_symbol:Pparg"
        - "rows": number of results to return (int). Default to 10 unless 
                  the user specifies otherwise.
        - "fl": comma-separated string of fields to return. 
                Example: "marker_symbol,marker_name,marker_description"

CORE-SPECIFIC RULES and EXAMPLE CALLS::
    Follow rules according to the requested solr core
    genotype-phenotype: [genotype-phenotype](./impc-api-query/references/genotype-phenotype-rules.md)
    experiment: 
    impc_images:
    phenodigm: [phenodigm](./impc-api-query/references/phenodigm-rules.md)
    gene:
    mp:
    pipleine:
    product:
    statistical-result:


RUNNING A QUERY:
With the core and params that you have gathered use the rules and example calls to execute the following:
    
``` uv run
    from impc_api import solr_request
    _, df = solr_request(core, params, silent=True)
    return df.to_dict(orient="records")
```

    
           
