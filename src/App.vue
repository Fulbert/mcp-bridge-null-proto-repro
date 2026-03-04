<script setup lang="ts">
/**
 * Minimal reproduction for https://github.com/hypothesi/mcp-server-tauri/issues/9
 *
 * The MCP bridge's unhandledrejection handler crashes with
 * "Cannot convert object to primitive value" when the rejection reason
 * is a null-prototype object (as produced by Vite/Rollup on import errors).
 *
 * Bug 1: String(reason) throws on null-prototype objects (no toString)
 * Bug 2: args.map(String) in captureLog also throws on null-prototype objects
 */

// Bug 1a: Simple null-prototype object — JSON.stringify works,
// but the real error metadata is lost (no "message" or "url" in output)
function triggerNullProtoRejection() {
  const nullProtoObj = Object.create(null);
  nullProtoObj.message = "Failed to fetch dynamically imported module";
  nullProtoObj.url = "/src/components/Missing.vue";
  Promise.reject(nullProtoObj);
}

// Bug 1b: Null-prototype object WITH circular reference —
// JSON.stringify fails, String() fails → bridge crashes with
// "Cannot convert object to primitive value"
function triggerNullProtoCircularRejection() {
  const nullProtoObj = Object.create(null);
  nullProtoObj.message = "Failed to fetch dynamically imported module";
  nullProtoObj.url = "/src/components/Missing.vue";
  // Add circular reference (common in Vite HMR error objects)
  nullProtoObj.self = nullProtoObj;
  Promise.reject(nullProtoObj);
}

// Bug 2: captureLog's args.map(String) throws on null-prototype objects
function triggerNullProtoConsoleLog() {
  const nullProtoObj = Object.create(null);
  nullProtoObj.error = "some error detail";
  // Add circular ref so JSON.stringify also fails
  nullProtoObj.self = nullProtoObj;
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

    <div class="row" style="gap: 1em; margin-top: 2em; flex-wrap: wrap;">
      <button @click="triggerNullProtoRejection">
        Bug 1a: Rejection (simple)
      </button>
      <button @click="triggerNullProtoCircularRejection">
        Bug 1b: Rejection (circular)
      </button>
      <button @click="triggerNullProtoConsoleLog">
        Bug 2: console.log
      </button>
    </div>

    <div style="margin-top: 2em; text-align: left; max-width: 600px; margin-inline: auto;">
      <h3>How to reproduce:</h3>
      <ol>
        <li>Connect tauri-mcp via <code>driver_session</code></li>
        <li>Click <strong>"Bug 1a"</strong> — JSON.stringify works but the error is an opaque JSON dump, not the original message</li>
        <li>Click <strong>"Bug 1b"</strong> — circular ref makes JSON.stringify fail, then String() crashes with <code>Cannot convert object to primitive value</code></li>
        <li>Click <strong>"Bug 2"</strong> — console.log with a null-prototype + circular object crashes <code>captureLog</code></li>
        <li>Read console logs after each click to observe the behavior</li>
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
