---
name: Playground
---

## Determine the directory

If the user specifies something like "in the current directory", assume it's a directory you can already play in.

Otherwise, determine the type (react, vue, svelte, livewire, etc) from the input and use the `scratchpad` command to spin up a new Laravel project. supply the name as something derived from the input + `-playground` (e.g. `svelte-echo-playground`).

The final command would look something like: `scratchpad svelte svelte-echo-playground`

The resulting directory is your new working directory, work from there.

## Link the library

Link the library or libraries mentioned into the project using pnpm link or composer local paths.

## Create the playground

Examine the specified library and explore all of its methods and options, create a robust playground that showcases its abilities.
