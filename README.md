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

# Set up uv - python package manager

For linux run 

curl -LsSf https://astral.sh/uv/install.sh | sh

Detailed instructions can be found here https://docs.astral.sh/uv/getting-started/installation/

# Build a server with Natural language

Now describe our server and what it needs to do. We will start with a simple file listing, and reading API.

PS you can set up the AutoApprove if you dont want Roo to stop and ask everytime it needs to take an action

- I want to build an mcp server using Typescript
- build the source code under a folder called mcp-file-server
- Server will allow access to local machine folders
- it should take a base folder name to work with. it cannot access any other location.
- It should work for three major operating systems, linux, Mac and windows
- it will show the list of files in the local drive
- it can read files from local drive

Paste this in the Roo console and wait. this will cost around US$2 for Claude and will take around 3mins to complete. You should have a new folder with the following file structure

mcp-file-server/
├── src/
│   └── index.ts          # Main server implementation
├── dist/                 # Compiled JavaScript (after build)
├── test-data/           # Test directory with sample files
│   ├── file1.txt
│   ├── file2.txt
│   └── subfolder/
│       └── nested.txt
├── package.json         # Node.js project configuration
├── tsconfig.json        # TypeScript configuration
├── README.md            # Comprehensive documentation
├── .gitignore          # Git ignore file
├── test-server.sh      # Test script to create sample data
└── mcp-config-example.json  # Example MCP client configuration


TODO: you can continue changing the code and update it as you wish. For example, add new functionality like writing a file.

Once you are happy with the code check it with the code I have created, "git checkout server-code" 

## Run MCP inspector to check if the server is working

we will run the inspector th check its compatibility for MCP. see details here https://modelcontextprotocol.io/docs/tools/inspector#inspecting-locally-developed-servers

go to where you run the code and run on the console

npx @modelcontextprotocol/inspector node path/to/server/index.js args...

for me it is npx @modelcontextprotocol/inspector node dist/index.js .

![Inspector Window](images/inspector-window.png)

See your server properties at http://127.0.0.1:6274, connect and list tools.

# Lets integrate our server to Roo.

Open Roo MCP config, third icon on the top after the "+" sign.

![alt text](images/roo-config.png)

Click Edit Global MCP and add the following to your config
WARNING: once you add the config you can stop the server that you have started from console. each agentic applicaiton in this case Roo will run its own instance behind the scenes.
{
  "mcpServers": {
    "file-server": {
      "command": "node",
      "args": ["/path/to/mcp-file-server/dist/index.js", "/path/to/base/folder"],
      "env": {}
    }
  }
}

mine looks like 

{
  "mcpServers": {
    "file-server": {
      "command": "node",
      "args": [
        "/home/daghan/git/blog_posts/handson-mcp/mcp-file-server/dist/index.js",
        "/home/daghan/git/blog_posts/handson-mcp/mcp-file-server/test-data"
      ],
      "env": {}
    }
  }
}

![alt text](images/roo-mcp-server.png)

if you don't get an error, it means it is now configured to use with Roo. You can also inspect the MCP tools in this window. if you want to know what to ask Roo what MCP tools it has access to using the follwing on the chat window.

@tools - list all available tools
Show me what MCP servers are connected
What external tools can you access?


# Building MCP client

Let's dig into defintions to clear up some of the concepts. @modelcontextprotocol/inspector is actually an applicition with a MCP client embedded in it. 

![alt text](images/Simple-mcp-application.png)

That is, client itslef does not have the smarts, it is just a client that connects to server. You can see a client code here https://github.com/modelcontextprotocol/quickstart-resources/blob/main/mcp-client-typescript/index.ts. Inspector application is a front end that controls the client and gets info about the server. To udnerstand it little better we will build our own client using Roo so we can see the bare bones. In theory this is a command line MCP application, but lets ignore this for the moment. 

Copy and paste the follwing code to the Roo chat.

- I want to build a MCP client
- Client code is stored under the folder mcp-client
- Client code is written in typescript
- it should take server executable and server parameters as input
- it should list the availabe tools on the MCP server on the console and it should terminate.

Warning, if you get error saying context exceeds your limit, just start a new task and paste the above code. This should cost you around US$1.5 and 2mins to complete.  Now you shouls have a project structure for your client as follows.

mcp-client/
├── src/
│   └── index.ts        # Main client implementation
├── dist/               # Compiled JavaScript (generated)
├── package.json        # Project configuration
├── tsconfig.json       # TypeScript configuration
├── README.md          # Documentation
└── .gitignore         # Git ignore file

In the readme file you can find how to run your client. Upon inspection it src/index.ts seems better qulaity than the official client code given here https://github.com/modelcontextprotocol/quickstart-resources/blob/main/mcp-client-typescript/index.ts.

if you want to run your client and see the server tools just copy past the follwing on your computer under the folder called mcp-client.

node dist/index.js node ../mcp-file-server/dist/index.js ../mcp-file-server/test-data


# MCP application with AI

So far we have built a MCP server and MCP client, but there is no intelligence in it. We will now create a chat MCP application which will list and read files for us. It iwll look like the image below

![alt text](images/complete-agentic-mcp-app.png)

We will use:
- Chainlit for chat app, I have copied read me from https://github.com/Chainlit/chainlit, it is called README-chainlit.md.
Strand agent as our agent (https://strandsagents.com/latest/), we do not need to build MCP client for strand since it already comes with the client. I have already copied read me from https://github.com/strands-agents/sdk-python. it is called README-strand-agent.md.
we will use our good old file server as our MCP server.

In this case you can use 2 methods to build the code. You need uv installed for this section

## Build code from high level diagram

- In Roo paste the image images/complete-agentic-mcp-app.png

Copy paste the the following instructions
- file chat app uses chainlit
- it should use Strand agent library as captured in README-strand-agent.md
- Strand agent needs to use Claude 4 optus LLM. 
- store code files under file-chat-app-from-diagram
- uise UV as python package manager
- only use README-chainlit.md, README-strand-agent.md and mcp-file-server as context. strictly do not use any other file.

If all goes well you will have a folder called file-chat-app-from-diagram and once you update the .env file (see the generated README.md) with your antropic key, run the command 

./run.sh

you can interact with your chat app on http://localhost:8000/

PS: In my case, the code did not use Strand python lib initially so I have used multiple prompts to convince Roo to use the Strand library :), Also, it has not utilizex "uv" for managing .venv and app running. I needed to tell Roo to correct those as well.


## Build code from description 

Write the follwing to the Roo chat

- I want to build a caht application using chainlit framework
- it should use Strand agent library as captured in README-strand-agent.md
- Strand agent needs to use Claude 4 optus LLM. 
- it should use mcp-file server
- store code files under file-chat-app-from-description
- use UV as python package manager
- only use README-chainlit.md, README-strand-agent.md and mcp-file-server as context. strictly do not use any other file.

if everything goes well then you will have file-chat-app-from-description. Once you update the .nv environment you can run the followng (see the README.md in file-chat-app-from-description)

./run.sh

