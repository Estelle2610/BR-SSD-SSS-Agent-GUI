<script>
  import Header from './Header.svelte';
  import WMR2Excel from './WMR2Excel.svelte';
  import SqlAgent from './SqlAgent.svelte';
  import Router from "svelte-spa-router";

  // Define routes
  const routes = {
    "/": WMR2Excel,    // Default page
    "/sqlagent": SqlAgent // Sql query page
  };
</script>

<Header />

<Router {routes} />

<style>
  body {
    margin: 0;
    font-family: Arial, sans-serif;
  }
</style>