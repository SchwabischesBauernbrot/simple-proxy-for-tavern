# Fake OpenAI API for Kobold

_A workaround to have more control over the prompt format when using SillyTavern and local models._

This script sits between SillyTavern and a backend like Kobold and it lets you change how the final prompt text will look. By default, it includes a prompt format that works well with LLaMA models tuned to follow instructions. It does this by presenting itself to SillyTavern as an OpenAI API, processing the conversation, and sending the prompt text to the backend.

The LLaMA tokenizer needs a modern Node.js version to work. Use the latest **LTS** version of Node.js.

You need a local backend like [KoboldAI](https://github.com/0cc4m/KoboldAI), [koboldcpp](https://github.com/LostRuins/koboldcpp) or [Ooba in API mode](https://github.com/oobabooga/text-generation-webui) to load the model, but it also works with the [Horde](http://koboldai.net/), where people volunteer to share their GPUs online.

## Table of Contents

- [Installation](#installation)
  - [Tavern Settings](#tavern-settings)
    - [Configuration File](#configuration-file)
    - [Manual](#manual)
  - [Notes](#notes)
- [File Structure](#file-structure)
- [Examples](#examples)
- [Changelog](#changelog)

## Installation

Clone this repository anywhere on your computer and run this inside the directory:

```sh
npm install
node index.mjs
```

You can replace the last line with this if you want it to reload automatically when editing any file:

```sh
npx nodemon index.mjs
```

Copy the file **config.default.mjs** to **config.mjs** if you want to make changes to the config. That way they aren't lost during updates.
If you're going to use the Horde, set your key and the models you want to use there.

There are now generation and prompt formats presets in the _presets/_ and _prompt-formats/_ folders.

### Tavern Settings

Download [alpaca.settings](./img/alpaca.settings) and put it in SillyTavern/public/OpenAI Settings/ and reload or start Tavern. Some of the values in the next steps will already be complete.

After pressing the second button of the top panel, select "OpenAI" as the API and write a random API key; it doesn't matter.
![api connections](./img/api.png)

Press the first button and select the "alpaca" preset. If it doesn't exist, create one. In older versions, the button might be at the bottom of that panel or to the right of the select box.

- Scroll up and set "OpenAI Reverse Proxy" to http://127.0.0.1:29172/v1
- Delete the default Main Prompt and NSFW Prompt.
- Change Jailbreak Prompt to "{{char}}|{{user}}". If you want to add your own text there, do it on the second line.
- Change Impersonation Prompt to "IMPERSONATION_PROMPT".
- On the checkboxes above, enable NSFW Toggle.
- Enable Streaming too if you want that.

![settings screenshot](./img/settings.png)

Press the second button from the top panel again and select "Connect".

### Notes

Leave Context Size high so Tavern doesn't truncate the messages, we're doing that in this script.

Tavern settings like Temperature, Max Response Length, etc. are ignored. Edit _generationPreset_ in conf.mjs to select a preset. The presets are located in the presets/ directory.
There's also a _replyAttributes_ variable that, by default, alters the prompt to induce the AI into giving more descriptive responses.

If you want to always keep the example messages of the character in the prompt, you have to edit _keepExampleMessagesInPrompt_ in conf.mjs while also enabling the option in the Tavern UI.

The last prompt is saved as prompt.txt. You can use it to check that everything is okay with the way the prompt is generated.

Streaming works for ooba and koboldcpp. Kobold doesn't support streaming or stopping strings.

Ooba needs to be started with --extensions api and the streaming API was added Apr 23, 2023.

## Files

- **config.default.mjs**: default settings
- **config.mjs**: user settings, if exists
- **index.mjs**: proxy code
- **horde.mjs**: horde code
- **presets/\*.json**: AI generation presets. The defaults come from Kobold.
- **prompt-formats/\*.mjs**: functions to build the prompt
- **tokenizer.model**: LLaMA tokenizer model from huggingface.

## Examples

[Rentry with examples from /lmg/](https://rentry.org/llama-examples)
![rp example](./img/example.jpg)

## Changelog

### 2023-05-02
- Reverted "add support to set the character names in the main prompt." That prompt is not sent when using impersonation. Changed it back to the first line of the Jailbreak.
- Added an option to include the character bias in the final text generated. It's enabled by default.
- Fixed how the singleline prompt format finds who sent the last message and added an option to customize the "[says nothing]" message.

### 2023-05-02
- Added Horde support, see config.default.mjs.
- Added character bias (a string added at the very end of the prompt)
- Added different configuration variable to set the max amount of tokens to generate while using impersonation.
- Added support to set the character names in Main Prompt in the first line with this format "{{char}}|{{user}}", freeing the jailbreak. The following lines after the first one can be used normally.

### 2023-04-29

- Added a config.mjs file for the settings.
- Added presets/ for generation presets and prompt-formats/ for the functions that generates the prompts.
