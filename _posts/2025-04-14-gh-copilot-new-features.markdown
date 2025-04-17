---
layout: post
title: "Celebrating Microsoft's 50th Anniversary with exciting new GitHub Copilot Features"
description: "Explore the latest GitHub Copilot features that enhance developer productivity and collaboration, including AI pair programming and code generation."
date: 2025-04-14 15:18:00 +0300
categories: Development GitHub
tags: development github github-copilot vscode
---

On April 4, 1975, Bill Gates and Paul Allen founded Microsoft. This month, we celebrate Microsoft's 50th anniversary. To mark this milestone, GitHub Copilot has rolled out several exciting new features designed to boost developer productivity and collaboration. In this post, we'll take a closer look at these new enhancements and discuss how they can streamline your development workflow. Additionally, we'll revisit some older features that have been updated or have become increasingly relevant.

## Agent Mode

Agent Mode is a new feature that allows GitHub Copilot to act as an intelligent assistant, providing context-aware suggestions. This mode enhances the coding experience since it allows the copilot to autonomously complete tasks. Up to now, this was only available in the VS Code Insiders edition, but now it is available in the regular version of VSCode.

### How to enable it

Agent Mode might not be enabled in your VSCode instance by default. To enable it, go to the VSCode settings (`CTRL+SHIFT+P` and search for `settings`) and search for `"Copilot Chat"`. You should see an option to enable it, as shown below.

![enable agent mode](/images/gh-copilot-199/01-enable-agent.png){: width="600" height="110" }

Once you enable it you can access it by opening the Copilot Chat panel and then selec one of the three available options: `Ask`, `Multi Edit`, or `Agent Mode`, as shown below:

![open copilot chat](/images/gh-copilot-199/02-copilot-chat-options.jpg){: width="400" height="120" }

You can also use the shortcut `CTRL+ALT+I` to open the Copilot Chat panel, and then with the shortcut `CTRL+.` you can switch between the three modes.

### Differences between Ask / Edit / Agent mode

- **Ask Mode:** This mode is designed for asking questions and getting quick answers. You can ask Copilot about specific code snippets, programming concepts, or general coding advice. The AI will provide concise responses to help you understand the topic better.
- **Edit Mode:** In this mode, you can provide Copilot with a code snippet or a specific task, and it will generate code based on your input. This is useful for generating boilerplate code or getting suggestions for implementing specific features. In edit mode, we give VSCode permission to edit and modify files, and the AI can suggest changes to multiple lines of code at once. This is useful for refactoring, for implementing a new feature or making bulk changes to a codebase.
- **Agent Mode:** This mode is more advanced and allows Copilot to act as a pair programming partner. It can analyze your code, suggest improvements, and even help you debug issues. Agent Mode is designed to provide a more interactive and collaborative coding experience, making it easier to work with complex codebases. It can ask you to go ahead and run commands in the terminal in order to install dependencies, run tests, or even deploy your code. This mode is particularly useful for developers who want to leverage AI assistance throughout the entire development process.

## How to interact with the GH Copilot

There are several ways to interact with GH Copilot. You can use the following shortcuts:

- Press `CTRL+ALT+I` to open the Copilot Chat panel.
- Press `Ctrl+Shift+Alt+L` to open the Quick Chat and ask a question (only "ask mode").
- Place the cursor on a line of code and press `CTRL+I` to open the Inline Chat to send a chat request to Copilot directly from the editor. This is useful for asking questions about specific lines of code or getting suggestions for improvements. The Inline Chat feature allows you to interact with Copilot without leaving the editor, making it easier to get help while coding.

Additionally you can use _smart actions_ to get help from Copilot without having to write a prompt. Examples of these smart actions are generating commit messages, generating documentation, explaining or fixing code, or performing a code review. These smart actions are available throughout the VS Code UI usually with an icon that looks like a _magic wand_ or two small stars. An example of a commit message generation is shown below:
![commit message example](/images/gh-copilot-199/commit-message-example.jpg){: width="300" height="100" }

## GH Copilot Chat participants

Chat participants are AI-powered "domain experts" you can tag in Copilot Chat to get context-specific help. There are several participants available, including:

- **@workspace**: Knows your entire open project in VS Code. It can answer questions about your codebase, like finding where a function is defined or suggesting improvements based on your project's structure and files.
- **@github**: Pulls in context from your GitHub repositories, issues, pull requests, and (if enabled) web searches via Bing. It's useful for questions tied to your GitHub projects or broader coding queries that need external context.
- **@terminal**: Understands the integrated terminal in VS Code, including its commands, output, and history. It can help explain terminal errors or suggest commands to run.
- **@vscode**: Experts in VS Code itself. It can guide you on editor features, settings, extensions, or commands, like how to configure debugging or disable the minimap.
- **@azure**: Provides expertise on Azure services and tools. It can assist with Azure-specific tasks, like setting up cloud resources, deploying apps, or troubleshooting Azure-related code. To use that, you need to install the [Azure Copilot extension](https://marketplace.visualstudio.com/items/?itemName=ms-azuretools.vscode-azure-github-copilot).

### Copilot Chat send options

As you see in the image below, there are three options to send a message to the chat. The first one is the default one (called "Send and dispatch" or by just pressing `enter`), and it will detect if there are chat participants involved, so it will send the prompt to all chat participants. The second one is the simple "Send" option (`SHIFT+ALT+ENTER`), which will send the message just to GitHub Copilot, without involving any other participants. The third option is the `send with #codebase` (`CTRL+ENTER`), which will send the message to the GitHub Copilot and the @codebase participant.

![send options](/images/gh-copilot-199/03-send-options.jpg){: width="400" height="90" }

## GH Copilot tools and context (#commands)

With the `#keyword-or-file` command you can provide the copilot with additional context. For example you can point to a specific file in your workspace by adding `#filename` command.

More tools added recently:

- `#fetch`: It is used to provide it with a URL to fetch documentation from. This is useful when you want to provide the agent mode with more specific information about your project or when you want it to fetch documentation from a specific URL, such as a README file or an API documentation page. This can help the copilot to provide more relevant and accurate suggestions especially in the case of really fresh APIs/technologies that have been created after the cutoff date of the LLM supporting the copilot.
- `#usages`: This is a combination of "Find All References", "Find Implementation", and "Go to Definition". This tool can help chat to learn more about a function, class, or interface. For instance, chat can use this tool to look for sample implementations of an interface or to find all places that need to be changed when making a refactoring.

The context choices are greatly increased with several VSCode extensions that are available in the marketplace. Alternatively you can select the "Add Context" option from the chat menu, which will open a list of available extensions. You can then select the extension you want to use and it will be added to the chat context, as shown below.

![add context](/images/gh-copilot-199/04-add-context.jpg){: width="400" height="350" }

### Advanced Codebase Search

You can use `#codebase` to provide the copilot with the context of your codebase. If you use that in your query, Copilot will help you find relevant code in your workspace for your chat prompt, since it will run tools like text search and file search to pull in additional context from your workspace. To enable that you need to set in your settings the `github.copilot.chat.codesearch.enabled` to `true`.

## Choose the right LLM

The new GitHub Copilot features allow you to choose the right LLM (Large Language Model) for your specific needs. This means you can select from a variety of models, each optimized for different tasks and programming languages. This flexibility allows developers to tailor their AI assistance to better suit their workflow and project requirements. However what model you choose has also a financial dimension that you need to consider.

At the time of writing this post, the base model is GPT-4o, and hence its usage has no financial limitation. For the most of the paid versions of GitHub Copilot, you can get a number of _Premium Requests_ per month. For instance, with the most common paid plan (i.e. [_Copilot Business_](https://docs.github.com/en/copilot/about-github-copilot/subscription-plans-for-github-copilot#comparing-copilot-plans)), you can receive up to 300 _Premium Requests_ monthly. Once you run out of _Premium Requests_, you will be [charged $0.04 per request for the additional requests](https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests#additional-premium-requests).

But not every _Premium Model_ you will use will consume the same amount of the available _Premium Requests_. For instance, the `Claude 3.7 Sonnet` model will consume 1 _Premium Request_ per request, while the `Claude 3.7 Sonnet Thinking` model will consume 1.25 _Premium Requests_ per request. More details on that you can find in the [Model multipliers documentation](https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests#model-multipliers)

### Bring your own LLMs (model management)

However, you are now given the option to bring your own LLMs. This means you can integrate your preferred LLMs into GitHub Copilot, for popular providers such as Azure, Anthropic, Gemini, Open AI, Ollama, and Open Router. This allows you to use new models that are not natively supported by Copilot the very first day that they're released.

## MCP Servers in Agent mode

One of the most exciting new features of GitHub Copilot is the support for **Model Context Protocol (MCP)** servers in agent mode, allowing AI models to interact with external tools, applications, and data sources dynamically. MCP servers can be configured within VS Code settings, supporting input variables for secure and flexible setups. Users can add new MCP servers via command-line tools or AI-assisted setup, and the servers start on demand to optimize resource usage. Additionally, built-in tools in agent mode enable enhanced functionalities such as fetching web content and performing deep code analysis. Additionally, these tools can assist in automating repetitive tasks and improving overall coding efficiency.

## Next Edit Suggestions

Next Edit Suggestions is a new feature that allows GitHub Copilot to suggest the next line of code based on the current context. This feature is designed to help developers write code faster and more efficiently by providing real-time suggestions as they type. Next Edit Suggestions can be particularly useful for repetitive tasks or when working with complex code structures.

## Copilot vision

End-to-end vision support is still in preview, but greatly improved in this version of Copilot Chat. Using the GPT4o base model, you can attach images and interact with images in chat prompts. For example, if you attach a screenshot of VS Code with some error, you can ask Copilot to help you resolve the issue. You might also use it to attach some UI mockup and let Copilot provide some HTML and CSS to implement the mockup.

## Final Thoughts

GitHub Copilot has introduced several new features that enhance the developer experience and improve productivity. The new Agent Mode, advanced codebase search, and the ability to bring your own LLMs are just a few of the exciting updates that make GitHub Copilot an even more powerful tool for developers. As we celebrate Microsoft's 50th anniversary, it's clear that the company is committed to pushing the boundaries of technology and empowering developers to achieve more.

In our next blog post, we'll dive deeper into Agent Mode and provide tips on how to use it effectively.

## References

- [VSCode 1.99 release](https://code.visualstudio.com/updates/v1_99)
- [Model multipliers documentation](https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests#model-multipliers)
- [Copilot Plans](https://docs.github.com/en/copilot/about-github-copilot/subscription-plans-for-github-copilot#comparing-copilot-plans)
