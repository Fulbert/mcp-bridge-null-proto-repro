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
        <li>Run <code>pnpm tauri dev</code> — devtools open automatically</li>
        <li>Click the button above and check the Console tab</li>
        <li>
          With the bridge enabled (line 11 in <code>src-tauri/src/lib.rs</code> uncommented),
          you get an error instead of the log:<br>
          <code>Uncaught TypeError: Cannot convert object to primitive value</code>
        </li>
        <li>
          Comment out line 11 and save — Tauri restarts automatically.
          Click again and the <code>console.log</code> works normally with no crash.
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
