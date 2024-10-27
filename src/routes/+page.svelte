<script lang="ts">
  import { Command } from "@tauri-apps/plugin-shell";

  let application = "cmd";
  let args = "";
  let textAreaValue = "";

  async function getOutput(application: string, args: string | string[]) {
    textAreaValue = "Loading...";
    return new Promise(async (resolve) => {
      let output: string = "";
      const command = Command.create(application, args);
      command.on("close", () => resolve(output));
      command.on("error", (error) => resolve("ERROR:\n" + String(error)));
      command.stdout.on("data", line => output += line);
      command.stderr.on("data", line => output += line);
      await command.spawn();
    });
  }
</script>

<main class="container">
  <h1>Enter a CMD or PowerShell command</h1>
  <div class="row">
    <select bind:value={application}>
      <option value="cmd">cmd</option>
      <option value="powershell">powershell</option>
    </select>
    <input placeholder='Arguments or ["arg", "arg", ...]' style="width: 70vh;" bind:value={args} required on:keyup={async (evt) => {
      if (evt.key === "Enter" && args.length > 0) { textAreaValue = String(await getOutput(application, args.startsWith("[") && args.endsWith("]") ? JSON.parse(args) : args)); }
    }} />
    <button disabled={args.length === 0} on:click={async () => textAreaValue = String(await getOutput(application, args.startsWith("[") && args.endsWith("]") ? JSON.parse(args) : args))}>Spawn command</button>
  </div>
  <div class="row" style="margin-top: 10px;">
    <textarea bind:value={textAreaValue} style="width: calc(100% - 20px); resize: none; height: calc(100vh - 165px);" rows="18" readonly></textarea>
  </div>
</main>

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
    display: flex;
    flex-direction: column;
    text-align: center;
    height: calc(100vh - 1rem);
  }

  .row {
    display: flex;
    justify-content: center;
  }

  h1 {
    text-align: center;
  }

  input,
  button {
    border-radius: 8px;
    border: 1px solid transparent;
    padding: 0.6em 1.2em;
    font-size: 1em;
    font-weight: 500;
    font-family: inherit;
    color: #0f0f0f;
    margin: 5px;
    background-color: #ffffff;
    transition: border-color 0.25s;
    box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
  }

  select,
  textarea {
    border-radius: 8px;
    border: 1px solid transparent;
    padding: 5px 10px;
    margin: 5px;
    font-size: 1em;
    color: #0f0f0f;
    background-color: #ffffff;
    transition: border-color 0.25s;
    box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
  }

  button {
    cursor: pointer;
  }

  button:hover {
    border-color: #396cd8;
  }
  button:active {
    border-color: #396cd8;
    background-color: #e8e8e8;
  }

  button:disabled {
    color: grey;
    border: 1px solid transparent;
  }

  input,
  button {
    outline: none;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      color: #f6f6f6;
      background-color: #2f2f2f;
    }

    input,
    button,
    select,
    textarea {
      color: #ffffff;
      background-color: #0f0f0f98;
    }
    button:active {
      background-color: #0f0f0f69;
    }
  }
</style>
