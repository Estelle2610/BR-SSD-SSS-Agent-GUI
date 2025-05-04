<script>
    import AgentIcon from '../Agent.ico';
    import { location } from "svelte-spa-router";
  </script>
  
  <header>
    <div class="logo">     
      <span>BR/SSD/SSS Assistant</span>
    </div>
    <div class="icon-container">
      <img src={AgentIcon} alt="Agent Icon" class="icon" />
    </div>
    <nav>
      <a href="#/" class={$location === "/" ? "active" : ""}>WMR (11.48)</a> |
      <a href="#/sqlagent" class={$location === "/sqlagent" ? "active" : ""}>DB Query</a>
    </nav>
  </header>
  
  <style>
    header {
      background: white;
      color: black;
      padding: 1rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
  
    .logo {
      display: flex;
      align-items: center;
      /* gap: 0.5rem; */
    }
  
    .logo span {
      font-size: 1.2rem;
      font-weight: bold;
    }

    .icon {
      width: 60px;
      height: 60px;
      object-fit: scale-down;
      margin-right: 40px;
      margin-left: 40px;
    }
    a {
    text-decoration: none;
    color: black;
    padding: 5px;
  }
  .active {
    text-decoration: underline;
    font-weight: bold;
    color: black;
  }
  </style>