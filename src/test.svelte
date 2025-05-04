<!-- TestComponent.svelte -->
<script>
    import  saveAs from 'file-saver';
    
    const testSave = () => {
      const blob = new Blob(["Hello world"], {type: "text/plain"});
      saveAs(blob, "test.txt");
    }
  </script>
  
  <button on:click={testSave}>Test File-Saver</button>