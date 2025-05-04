<script>
    // import { onMount } from 'svelte';
    export const name  = "QueryDB";
  
    let selectedDatabase = '';
    let selectedQuery = '';
    let workflowDescription = ''; 
    let workflows = [];
    let dbType = "sns";
    let dbPath = "";
    let errorMessage = "";
  
    // Fetch workflows based on the selected database
    async function fetchWorkflows() {
      if (dbType) {
        const response = await fetch(`http://127.0.0.1:8000/workflows?db=${dbType}`);
        workflows = await response.json();
      }
    }
  
    // Fetch description for the selected workflow
    async function fetchWorkflowDescription(workflowId) {
      if (workflowId) {
        const response = await fetch(`http://127.0.0.1:8000/workflows/description?id=${workflowId}`);
        const data = await response.json();
        workflowDescription = data.description || '';
      }
    }
  
    // Handle query execution
    async function executeQuery() {
      const response = await fetch('http://127.0.0.1:8000/execute_query', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          database: selectedDatabase,
          query: selectedQuery
        }),
      });
  
      const result = await response.blob();
      const downloadLink = document.createElement('a');
      downloadLink.href = URL.createObjectURL(result);
      downloadLink.download = 'query_results.xlsx';
      downloadLink.click();
    }
  </script>

<div class="querydb-container">  
    <div >
      <h2>SNS SNTrack Query Help</h2>
      
      <!-- Database connection -->
      <div class="form-group">
        <label for="dbType">Database Type:</label>
        <select id="dbType" bind:value={dbType} on:change="{fetchWorkflows}">
          <option value="sns">SNS</option>
          <option value="sntrack">SNTrack</option>
        </select>
      </div>
          
      {#if selectedDatabase}
            <label for="workflow-select">Select Workflow:</label>
            <select id="workflow-select" bind:value={selectedQuery} on:change="{() => fetchWorkflowDescription(selectedQuery)}">
            <option value="">Select a workflow</option>
            {#each workflows as workflow}
                <option value={workflow.id}>{workflow.name}</option>
            {/each}
            </select>

            {#if workflowDescription}
            <div>
                <h3>Workflow Description:</h3>
                <p>{workflowDescription}</p>
            </div>
            {/if}

            {#if selectedQuery}
            <button on:click={executeQuery}>Query</button>
            {/if}
      {/if}

      <div class="form-group multiline">
        <label for="databaseFile">Database Path:</label>
        <textarea 
          id="databaseFile"
          bind:value={dbPath} 
          placeholder="Database Path"
          rows="2"          
        ></textarea>      
      </div>
          
      {#if errorMessage}
        <div class="error">{errorMessage}</div>
      {/if}
    </div>
   
  </div>
  
  <style>
    select, button {
      margin: 10px;
    }
    .querydb-container {
    max-width: 700px;
    margin: 2rem auto;
    padding: 2rem;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
  </style>
  