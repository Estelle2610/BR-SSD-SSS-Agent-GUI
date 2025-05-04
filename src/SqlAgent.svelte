
<script>
  export const name  = "SqlAgent";
  
  // 组件状态
  let dbType = "sns";
  let dbPath = "";
  let selectedQuery = "TOTO";
  let connectionStatus = "disconnected";
  let results = null;
  let isLoading = false;
  let isConnecting = false;
  let isDisconnecting = false;
  let errorMessage = "";
  let sqlQuery = "";
  let workflows = [];
  let workflowDescription = "workflow descrition blablabla...";
  import * as XLSX from 'xlsx';


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

  async function executeQuery() {
    const response = await fetch('http://127.0.0.1:8000/execute_query', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        db_type: 'SNTrack',
        query_id: 'API_monthly_statistics'
      }),
    });

    let queryResult = [];
    if (response.ok) {
      queryResult = await response.json();
    } else {
      console.error('Failed to fetch query result');
    }

    const ws = XLSX.utils.json_to_sheet(queryResult);  
    const wb = XLSX.utils.book_new();                  
    XLSX.utils.book_append_sheet(wb, ws, 'Data');      
    XLSX.writeFile(wb, 'query_result.xlsx');           
    console .log('Query result exported to query_result.xlsx');
  }
    
</script>

<div class="sqlagent-container">  
  <div class="connection-section">
    <h2>SNS SNTrack Query Help</h2>
    
    <!-- Database connection -->
    <div class="form-group">
      <label for="dbType">Database Type:</label>
      <select id="dbType" bind:value={dbType} on:change="{fetchWorkflows}">
        <option value="sns">SNS</option>
        <option value="sntrack">SNTrack</option>
      </select>
    </div>    

    <div class="form-group multiline">
      <label for="databaseFile">Database Path:</label>
      <textarea 
        id="databaseFile"
        bind:value={dbPath} 
        placeholder="Database Path"
        rows="2"
        disabled={connectionStatus === 'connected'}
      ></textarea>      
    </div>    
    
    <div class="form-group">
      {#if dbType}
        <label for="workflow-select">Select Query:</label>
        <select id="workflow-select" bind:value={selectedQuery} on:change="{() => fetchWorkflowDescription(selectedQuery)}">
          <option value="">Select a workflow</option>
          {#each workflows as workflow}
            <option value={workflow.id}>{workflow.name}</option>
          {/each}
        </select>
      {/if}
    </div>
        {#if workflowDescription}
          <div class="description-section">          
            <p>{workflowDescription}</p>
          </div>
        {/if}        
    
    <div class="button-container">
       <button 
         on:click={executeQuery}
         disabled={!selectedQuery}
         class="query-button"
       >
         Query
       </button>
    </div>    
  </div>  

  {#if errorMessage}
      <div class="error">{errorMessage}</div>
  {/if}
</div>

<style>
  .sqlagent-container {
    max-width: 700px;
    margin: 2rem auto;
    padding: 2rem;
    background-color: #c2c0a2; /* Light yellow paper color */
    box-shadow: 
      0 2px 8px rgba(0,0,0,0.1),
      inset 0 0 30px rgba(0,0,0,0.05); /* Inner shadow for paper effect */
    border: 1px solid #e6d595;
    border-radius: 2px;
    position: relative;
  }  
  h2 {
    margin-top: 0;
    margin-bottom: 2rem;
  }
  .form-group {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 1rem;
    margin-bottom: 1rem;
  }
  select {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 14px;
  }
  .form-group.multiline {
    grid-template-rows: 1fr;
  }   
  .error {
    color: #dc3545;
    margin-top: 1rem;
    padding: 0.5rem;
    background: #fff3f3;
    border-radius: 4px;
  }
  textarea:disabled {
    background-color: #e9ecef;
    cursor: not-allowed;
    opacity: 0.8;
  }
  .button-container {
    display: flex;
    justify-content: center;
    margin-top: 2rem;
    padding-top: 1rem;
    border-top: 1px solid #e6d595;
  }
  .query-button {
    min-width: 120px;
    background: #007bff;
    color: white;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .query-button:disabled {
    background: #6c757d;
    cursor: not-allowed;
    opacity: 0.65;
  }
  .description-section {
    margin-bottom: 1rem;
  }
</style>