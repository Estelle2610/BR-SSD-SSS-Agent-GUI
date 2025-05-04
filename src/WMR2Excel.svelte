<script>
   export const name = "WMR2Excel";

  let excelFileName = '11_48.xlsx';
  let sheetName = 'WMR_table';
  let wordFile = null;
  let fileExists = false;
  let isProcessing = false;
  let ific_no = 'xxxx';
  const today = {
    date: new Date().toISOString().split('T')[0] // Format: YYYY-MM-DD
  };

  // Check if file exists in backend
  const checkFileExists = async () => {
    try {
      const response = await fetch('http://localhost:5000/api/check-file', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
        body: JSON.stringify({ excel_name: excelFileName })        
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const data = await response.json();
      fileExists = data.exists;
      
      if (fileExists) {
        const overwrite = confirm(`File ${excelFileName} exists. Overwrite?`);
        if (!overwrite) {
          const newName = prompt('Enter new filename:', excelFileName);
          if (newName) excelFileName = newName;
        }
      }
    } catch (error) {
      // alert(`Error checking file: ${error.message}`);
      console.error('Error details:', error);
      if (error.message === 'Failed to fetch') {
        alert('Cannot connect to server. Please make sure the backend is running on port 5000');
      } else {
        alert(`Error checking file: ${error.message}`);
      }
    }
  };

  // Handle file conversion
  const handleConvert = async () => {
    if (!wordFile) {
      alert('Please upload a Word document first');
      return;
    }

    isProcessing = true;
    
    try {
      const formData = new FormData();
      formData.append('file', wordFile);
      formData.append('excel_name', excelFileName);
      formData.append('sheet_name', sheetName);
      formData.append('indate', today.date);
      formData.append('ific_no', ific_no);

      const response = await fetch('http://localhost:5000/api/convert', {
        method: 'POST',
        body: formData
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || 'Conversion failed');
      }

      // // Trigger file download
      // const blob = await response.blob();
      // const url = window.URL.createObjectURL(blob);
      // const a = document.createElement('a');
      // a.href = url;
      // a.download = excelFileName;
      // a.click();
      // window.URL.revokeObjectURL(url);

      // Show success message instead of downloading
      alert('File converted successfully!');

    } catch (error) {
      alert(`Error: ${error.message}`);
    } finally {
      isProcessing = false;
    }
  };

  // Handle file input change
  const handleFileChange = (e) => {
    wordFile = e.target.files[0];
    if (wordFile) {
      // Auto-generate Excel filename
      const baseName = wordFile.name.replace(/\.[^/.]+$/, "");
      excelFileName = `${baseName}_11_48.xlsx`;
    }
  };

  // Reactive check when filename changes
  $: if (excelFileName) checkFileExists();
</script>

<div class="converter">
  <h2>Weekly Meeting Report (11.48) Processing</h2>
  
  <!-- File Upload -->
  <div class="form-group">
    <label for="wordFileInput">Upload Word Document:</label>
    <input 
      id="wordFileInput"
      type="file" 
      accept=".docx" 
      on:change={handleFileChange}
      disabled={isProcessing}
    />
  </div>

  <div class="form-row">
    <div class="form-group half">
        <label for="excelFileNameInput">Excel Filename:</label>
        <input
            id="excelFileNameInput"
            type="text"
            bind:value={excelFileName}
            disabled={isProcessing}
        />
    </div>

    <div class="form-group half">
        <label for="sheetNameInput">Sheet Name:</label>
        <input
            id="sheetNameInput"
            type="text"
            bind:value={sheetName}
            disabled={isProcessing}
        />
    </div>
  </div>

  <div class="form-row">
    <div class="form-group half">
        <label for="sharepointIndateInput">SharePoint INDate:</label>
        <input
            id="sharepointIndateInput"
            type="text"
            bind:value={today.date}
            disabled={isProcessing}
        />
    </div>

    <div class="form-group half">
        <label for="ificNoInput">BR IFIC No.:</label>
        <input
            id="ificNoInput"
            type="text" placeholder="to publish in which edition"
            bind:value={ific_no}
            disabled={isProcessing}
        />
    </div>
  </div>

  <button 
    on:click={handleConvert}
    disabled={!wordFile || isProcessing}
  >
    {isProcessing ? 'Processing...' : 'Submit'}
  </button>
</div>

<style>
 .converter {
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
    margin-bottom: 1.5rem;
  }
  label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
  }
  input[type="file"] {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    display: flex;
    gap: 1rem; /* Adds space between button and text */
  }
  input[type="file"]::-webkit-file-upload-button {
    margin-right: 1rem; /* Adds space after the button */
    padding: 0.5rem 1rem;
    background: #4f6c88;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  input[type="text"] {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
  }

  button {
    background: #007bff;
    color: white;
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background 0.3s ease;
  

  }

  button:disabled {
    opacity: 0.6;
    background: #6c757d;
    cursor: not-allowed;
  }

  .form-row {
        display: flex;
        gap: 1rem;
        margin-bottom: 1.5rem;
    }

  .half {
        flex: 1;
        margin-bottom: 0;
    }
</style>