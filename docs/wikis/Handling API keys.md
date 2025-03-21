# Handling API keys

## Generating GitHub API

Your GitHub token can be generated [on your GitHub setting](https://github.com/settings/tokens/new?scopes=repo). The correct settings should already be applied. To avoid generating a new token every few months, select the “No expiration” option. Click the “Generate token” button and copy the token you are presented with on the next page.

## Preventing exposing your API keys

To protect you from exposing API keys, GitHub will scan your public repo and prevent you to push if it finds one. If the key is a GitHub key, it will expire the key immediately, forcing you to generate a new one and adding it to Enveloppe again. To prevent this, you may want to store the key in a standalone file, and prevent it to be added when commiting. That standalone file is called `env`. By default it is in the Envenloppe path: `.obsidian/plugins/obsidian-mkdocs-publisher/env`

To prevent the keys to be added when committing, create a file `.gitignore` in the root of the vault with this content:

```gitignore
.obsidian/plugins/obsidian-mkdocs-publisher/env
.obsidian/plugins/obsidian-mkdocs-publisher/logs.txt
```

## Updating API keys in multiple vaults

In the case you use one same key in multiple vaults, and you have to update it, the below script will help you automate that. Install [Deno](https://deno.com/), and create a file named `Update GitHub API key.ts` with the content below, then run this in the terminal:

```PowerShell
deno run -A "Update GitHub API key.ts"
```

Here is the content of the script:

```ts
import { join } from "jsr:@std/path";

type AbsolutePath = string & { readonly isAbsolute: true };
const ALL_VAULT_FOLDER = "D:\\Quả Cầu\\Vaults" as AbsolutePath;
const API_GITHUB = "XXXXXXXXXXXXXXXXXXXX"

async function hasObsidianSetting(folder: AbsolutePath) {
  try {
    for await (const dirEntry of Deno.readDir(folder)) {
      if (dirEntry.isDirectory && dirEntry.name === ".obsidian") return true;
    }
    return false;
  }
  return false;
}

async function updateEnvEnveloppe(subfolder: AbsolutePath) {
const env = `GITHUB_TOKEN=${API_GITHUB}`
  const pathToEnvEnveloppe = join(subfolder, ".obsidian", "plugins", "obsidian-mkdocs-publisher", "env");
  try {
    await Deno.writeTextFile(pathToEnvEnveloppe, env);
  }
}

async function updateAllVault(folder: AbsolutePath = ALL_VAULT_FOLDER) {
  for await (const dirEntry of Deno.readDir(folder)) {
    const subFolder = join(folder, dirEntry.name) as AbsolutePath;
    if (await hasObsidianSetting(subFolder)) {
      await updateEnvEnveloppe(subFolder);
    } else if (dirEntry.isDirectory) {
      await updateAllVault(subFolder);
    }
  }
}

await updateAllVault();
```