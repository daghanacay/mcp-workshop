# Introduction

We are going to build a MCP server and integrate it into multiple agentic applications. MCP can be compared to REST API as discussed here https://docs.roocode.com/features/mcp/mcp-vs-api. In summary, REST APIs and MCP serve different tiers in the technology stack:

REST: Low-level web communication pattern that exposes operations on resources
MCP: High-level AI protocol that orchestrates tool usage and maintains context

MCP often uses REST APIs internally, but abstracts them away for the AI. Think of MCP as middleware that turns discrete web services into a cohesive environment the AI can operate within.

# Set up Roo

## Set up Roo plugin

Roo is our development envuroment that helps us building code. Roo is a generic tool that connects to LLM back end, in particular Claude to help speed up development. 

Open VS Code
Access Extensions: Click the Extensions icon in the Activity Bar or press Ctrl+Shift+X (Windows/Linux) or Cmd+Shift+X (macOS)
Search for "Roo Code"
Select "Roo Code" by RooVeterinaryInc and click Install
Reload VS Code if prompted

Please see details of setting up Roo here https://docs.roocode.com/getting-started/installing

## Set up LLM backend

In order to set up the AI(LLM) back end. We will use Anthropic here.

Go to console.anthropic.com
Sign up for an account or log in
Navigate to the API keys section and create a new key
Important: Copy your API key immediately as it won't be displayed again

Once you have your API key:

Open the Roo Code sidebar by clicking the Roo Code icon () in the VS Code Activity Bar
In the welcome screen, select your API provider from the dropdown
Paste your API key into the appropriate field.
In the drop down select any model you like, newer models will be more costly. I will use Claude 4.0 Opus, the most powerful and expensive yet.

please refer to Connec to AI backend https://docs.roocode.com/getting-started/connecting-api-provider for more details

Also turn off "Enable MCP server Creation" tag in the MCP server settings, sthird from left at the top of Roo code plugin interface. We will provide context for LLM in the foillwing setion.

## Set up knowdledge of MCP as contect for your LLM code platform

We are following Building MCP with LLMs - https://modelcontextprotocol.io/tutorials/building-mcp-with-llms

I have downlaoded the file for you in the source code llms-full.txt. this is the file that explins to your LLM how to build MCP server. On a different topic, Llms.txt is a new metadata standard to improve LLM serchability of web sites. https://llmstxt.org/ it is a large topic about SEO and you can do your research starting from https://searchengineland.com/llms-txt-proposed-standard-453676 one might think of a start up idea from this ;) but it is not the scope of this workshop.

we will use TS for this implementation so I have downloaded README doceumtn from https://github.com/modelcontextprotocol/typescript-sdk, file is saved as README-mcp-typescript.md

# Build a server with Natural language

Now describe our server and what it needs to do. We will start with a simple file listing, and reading API.

- I want to build an mcp server using Typescript
- build the source code under a folder called mcp-file-server
- Server will allow access to local machine folders
- it should take a base folder name to work with. it cannot access any other location.
- It should work for three major operating systems, linux, Mac and windows
- it will show the list of files in the local drive
- it can read files from local drive

Paste this in the Roo console and wait. this will cost around US$1 and will take around 5mins to complete. 

TODO: you can continue changing the code and update it as you wish. For example, add new functionality like writing a file.

Once you are happy with the code check it with the code I have created. 

we will run the inspector th check its compatibility for MCP. see details here https://modelcontextprotocol.io/docs/tools/inspector#inspecting-locally-developed-servers

go to where you ruin the code and run on the console

npx @modelcontextprotocol/inspector node path/to/server/index.js args...

for me it is npx @modelcontextprotocol/inspector node index.js ~/git

# Lets integrate our server to Roo.

Open Roo MCP config, third icon on the top after the "+" sign.
Click Edit Global MCP and add the following to your config
WARNING: once you add the config you can stop the server that you have started from console. each agentic applicaiton in this case Roo will run its own instance behind the scenes.
{
    "mcpServers": {
        "file-server": {
            "command": "node /ABSOLUTE/PATH/TO/PARENT/FOLDER/index.js",
            "args": [
                "~/git"
            ]
        }
    }
}

mine looks like 

{
  "mcpServers": {
    "file-server": {
            "command": "node /home/daghan/git/blogposts/handson-MCP/mcp-file-server/index.js",
            "args": [
                "~/git"
            ]
        }

  }
}

if you don't get an error then it means it is now configured to use with Roo. You can also ask Roo what MCP tools it has access to using the follwing on the chat window.

@tools - list all available tools
Show me what MCP servers are connected
What external tools can you access?


Client is just to connect to server
TODO but full code can be found here https://github.com/modelcontextprotocol/quickstart-resources/tree/maina