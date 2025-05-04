
<script>
  export const name  = "SqlAgent";

  import { onMount } from 'svelte';
  
  // 组件状态
  let dbType = "sns";
  let dbPath = "";
  // let username = "";
  // let password = "";
  let searchTerm = "";
  let queries = [];       // 从后端加载的查询列表
  let selectedQuery = null;
  let connectionStatus = "disconnected";
  let results = null;
  let isLoading = false;
  let isConnecting = false;
  let isDisconnecting = false;
  let errorMessage = "";
  let sqlQuery = "";
  
  onMount(async () => {
    try {
      const response = await fetch('/api/queries');
      queries = await response.json();
    } catch (err) {
      errorMessage = "无法加载查询列表";
    }
  });

  // 连接数据库
  const handleConnect = async () => {
    isConnecting = true;
    try {
      const response = await fetch('http://localhost:5000/api/connect', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ dbType, dbPath })
      });
      
      const data = await response.json();
      alert(data.message || data.error)
      connectionStatus = data.status;
      errorMessage = data.error || "";
    } catch (err) {
      errorMessage = "Connection failed";
      console.error("Connection failed:", err);
    }
    isConnecting = false;
  };

  const handleDisconnect = async () => {
    isDisconnecting = true;
    try {
      const response = await fetch('http://localhost:5000/api/disconnect', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ dbPath })
      });
      
      const data = await response.json();
      if (data.status === 'disconnected') {
        connectionStatus = 'disconnected';
        results = null;
        selectedQuery = null;
      }
      errorMessage = data.error || "";
    } catch (err) {
      errorMessage = "Disconnection failed";
      console.error("Disconnection failed:", err);
    }
    isDisconnecting = false;
  };

  // 执行查询
  const executeQuery = async () => {
    if (!sqlQuery.trim()) return;
    
    isLoading = true;
    try {
      const response = await fetch(`http://localhost:5000/api/execute_query`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ dbPath, sqlQuery })
      })
      .then(response => response.json())
      .then(data => {
          if (data.file_path) {
              window.open(data.file_path);  // Open the Excel file
          } else {
              console.error('Error:', data.error);
          }
      })
     .catch(error => console.error('Error:', error));
    } catch (err) {
      errorMessage = "Query execution failed";
    }
    isLoading = false;
  };
  //     });
  //     results = await response.json();
  //     errorMessage = "";
  //   } catch (err) {
  //     errorMessage = "Query execution failed";
  //   }
  //   isLoading = false;
  // };
</script>

<div class="sqlagent-container">  
  <div class="connection-section">
    <h2>SNS SNTrack Query Help</h2>
    
    <!-- Database connection -->
    <div class="form-group">
      <label for="dbType">Database Type:</label>
      <select id="dbType" bind:value={dbType}>
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

    <div class="button-group">
      <button 
        on:click={handleConnect}
        class:connected={connectionStatus === 'connected'}
        disabled={!dbPath || isConnecting || connectionStatus === 'connected'}
      >
      {isConnecting ? 'Connecting...' : connectionStatus === 'connected' ? 'Connected' : 'Connect'}
      </button>
    
      <button 
        on:click={handleDisconnect}
        class="disconnect-btn"
        disabled={isDisconnecting || connectionStatus !== 'connected'}
      >
      {isDisconnecting ? 'Disconnecting...' : 'Disconnect'}
      </button>
    </div>

    {#if errorMessage}
      <div class="error">{errorMessage}</div>
    {/if}
  </div>

  <!-- Database query execution -->
  {#if connectionStatus === 'connected'}
    <div class="query-section">
      <h2>Select SQL Query</h2>

      <div class="form-group multiline">
        <label for="sqlQuery">Enter SQL Query:</label>
        <textarea 
          id="sqlQuery"
          bind:value={sqlQuery} 
          placeholder="Enter your SQL query here..."
          rows="6"
        ></textarea>
      </div>
      
      

      <button 
        on:click={executeQuery} 
        disabled={!sqlQuery.trim() || isLoading}
        class="execute-btn"
      >
        {isLoading ? 'Executing...' : 'Execute Query'}
      </button>

      {#if results}
        <div class="results">
          <h3>Results</h3>
          <pre>{JSON.stringify(results, null, 2)}</pre>
        </div>
      {/if}
    </div>
  {/if}
</div>

<style>
  .sqlagent-container {
    max-width: 700px;
    margin: 2rem auto;
    padding: 2rem;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
  .form-group {
    margin: 1rem 0;
    display: grid;
    grid-template-columns: 120px 1fr;
    align-items: center;
    gap: 1rem;
  }
  select, input {
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
  button {
    min-width: 120px;
    background: #007bff;
    color: white;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: opacity 0.2s;
  }
  button.connected {
    background: #28a745;
  }
  button:disabled {
    opacity: 0.6;
    background: #6c757d;  
    cursor: not-allowed;    
  }
  .button-group {
    display: flex;
    justify-content: center;
    gap: 2rem;
    margin: 2rem 0;
  }
  .disconnect-btn {
    background: #dc3545;
  }
  .disconnect-btn:disabled {
    opacity: 0.6;
    background: #6c757d;
    cursor: not-allowed;
  }
  .query-list {
    margin: 1rem 0;
    border: 1px solid #eee;
    border-radius: 6px;
  }
  .query-item {
    display: flex;
    align-items: center;
    padding: 1rem;
    border-bottom: 1px solid #eee;
  }
  .query-item:hover {
    background: #f8f9fa;
  }
  .query-info {
    margin-left: 1rem;
  }
  .tags {
    margin-top: 0.5rem;
  }
  .tag {
    display: inline-block;
    background: #e9ecef;
    padding: 0.25rem 0.75rem;
    border-radius: 12px;
    font-size: 0.85em;
    margin-right: 0.5rem;
  }
  .results {
    margin-top: 2rem;
    padding: 1rem;
    background: #f8f9fa;
    border-radius: 6px;
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
</style>