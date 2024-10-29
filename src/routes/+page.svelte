<script lang="ts">
  import { onMount } from "svelte";
  import { resourceDir } from "@tauri-apps/api/path";
  import { Command } from "@tauri-apps/plugin-shell";

  let inputApp = "cmd";
  let inputArgs = "";
  let textAreaValue = "";

  onMount(() => resourceDir().then((res) => {
    textAreaValue = "Tauri Path API Loaded!\nExecutable folder:\n" + res;
  }).catch((err) => textAreaValue = "Error while loading Tauri Path API.\n" + err));

  async function getCommandOutput(application: string, args: string | string[]): Promise<string> {
    textAreaValue = "Loading...";
    let output = "";

    return new Promise((resolve) => {
      const timeoutId = setTimeout(() => resolve("Command timed out\nApplication: " + application + "\nArguments: " + JSON.stringify(args)), 5000);

      try {
        if (application === "powershell") {
          args = [
            "-ExecutionPolicy", "Bypass",
            "-NoProfile",
            "-Command", `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8; ${args}`
          ];
        }

        const command = Command.create(application, args);

        command.on("close", (data) => {
          clearTimeout(timeoutId);
          console.log("Command closed with code:", data.code);
          resolve(data.code === 0 ? output.trim() : "Command failed with code " + data.code);
        });

        command.on("error", (error) => {
          clearTimeout(timeoutId);
          console.error("Command error:", error);
          resolve("ERROR:\n" + String(error));
        });

        command.stdout.on("data", (line) => {
          console.log("stdout:", line);
          output += line.trimEnd() + "\n";
        });

        command.stderr.on("data", (line) => {
          console.error("stderr:", line);
          output += line.trimEnd() + "\n";
        });

        command.spawn();
      } catch (err) {
        clearTimeout(timeoutId);
        resolve("An error occurred:\n" + String(err));
      }
    });
  }
</script>

<main class="container">
  <h1>Enter a CMD or PowerShell command</h1>
  <div class="row">
    <select bind:value={inputApp}>
      <option value="cmd">cmd</option>
      <option value="powershell">powershell</option>
    </select>
    <input placeholder='Arguments or ["arg", "arg", ...]' style="width: 70vh;" bind:value={inputArgs} required on:keyup={async (evt) => {
      if (evt.key === "Enter" && inputArgs.length > 0) {
        textAreaValue = await getCommandOutput(inputApp, inputArgs.startsWith("[") && inputArgs.endsWith("]") ? JSON.parse(inputArgs) : inputArgs);
      }
    }} />
    <button disabled={inputArgs.length === 0} on:click={async () => {
      textAreaValue = await getCommandOutput(inputApp, inputArgs.startsWith("[") && inputArgs.endsWith("]") ? JSON.parse(inputArgs) : inputArgs);
    }}>Spawn command</button>
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
