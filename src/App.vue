<script setup lang="ts">
/**
 * Minimal reproduction for https://github.com/hypothesi/mcp-server-tauri/issues/9
 *
 * captureLog's args.map(String) throws on null-prototype objects with circular
 * references, crashing the bridge instead of forwarding the log.
 *
 * To observe the difference, toggle the MCP bridge in src-tauri/src/lib.rs:
 *   .plugin(tauri_plugin_mcp_bridge::init())  ← comment/uncomment this line
 */

function triggerNullProtoConsoleLog() {
  const nullProtoObj = Object.create(null);
  nullProtoObj.error = "some error detail";
  nullProtoObj.self = nullProtoObj; // circular ref — causes JSON.stringify to fail too
  console.log("Import failed:", nullProtoObj);
}
</script>

<template>
  <main class="container">
    <h1>MCP Bridge Null-Prototype Bug Repro</h1>
    <p>
      Issue:
      <a href="https://github.com/hypothesi/mcp-server-tauri/issues/9" target="_blank">#9</a>
    </p>

    <div class="row" style="margin-top: 2em;">
      <button @click="triggerNullProtoConsoleLog">
        Trigger: console.log with null-prototype object
      </button>
    </div>

    <div style="margin-top: 2em; text-align: left; max-width: 600px; margin-inline: auto;">
      <h3>How to reproduce:</h3>
      <ol>
        <li>
          Enable the MCP bridge in <code>src-tauri/src/lib.rs</code> (uncomment line 11),
          then connect via <code>driver_session</code>
        </li>
        <li>Click the button above</li>
        <li>
          Read console logs — instead of the log, you get:<br>
          <code>[MCP][BRIDGE][UNHANDLED_ERROR] Uncaught TypeError: Cannot convert object to primitive value</code>
        </li>
        <li>
          Now disable the bridge (comment line 11 back out) and repeat —
          the <code>console.log</code> works normally with no crash
        </li>
      </ol>
    </div>
  </main>
</template>

<style>
:root {
  font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;
  color: #0f0f0f;
  background-color: #f6f6f6;
}

.container {
  margin: 0;
  padding-top: 10vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}

.row {
  display: flex;
  justify-content: center;
}

button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  color: #0f0f0f;
  background-color: #ffffff;
  cursor: pointer;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
}

button:hover {
  border-color: #396cd8;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #f6f6f6;
    background-color: #2f2f2f;
  }
  button {
    color: #ffffff;
    background-color: #0f0f0f98;
  }
}
</style>
