# Docs MCP Server
[![smithery badge](https://smithery.ai/badge/@buger/docs-mcp)](https://smithery.ai/server/@buger/docs-mcp)

This project provides a flexible Model Context Protocol (MCP) server, powered by [Probe](https://probeai.dev/), designed to make documentation or codebases searchable by AI assistants.

You can chat with code or your docs, simply by pointing to git repo or a folder.
```
npx -y @buger/docs-mcp@latest --gitUrl https://github.com/buger/probe
```

**Use Cases:**

*   **Chat with any GitHub Repository:** Point the server to a public or private Git repository to enable natural language queries about its contents.
*   **Search Your Documentation:** Integrate your project's documentation (from a local directory or Git) for easy searching.
*   **Build Custom MCP Servers:** Use this project as a template to create your own official MCP servers tailored to specific documentation sets or even codebases.

The content source (documentation or code) can be **pre-built** into the package during the `npm run build` step, or configured **dynamically** at runtime using local directories or Git repositories. By default, when using a `gitUrl` without enabling auto-updates, the server downloads a `.tar.gz` archive for faster startup. Full Git cloning is used only when `autoUpdateInterval` is greater than 0.

## Features

- **Powered by Probe:** Leverages the [Probe](https://probeai.dev/) search engine for efficient and relevant results.
- **Flexible Content Sources:** Include a specific local directory or clone a Git repository.
- **Pre-build Content:** Optionally bundle documentation/code content directly into the package.
- **Dynamic Configuration:** Configure content sources, Git settings, and MCP tool details via config file, CLI arguments, or environment variables.
- **Automatic Git Updates:** Keep content fresh by automatically pulling changes from a Git repository at a configurable interval.
- **Customizable MCP Tool:** Define the name and description of the search tool exposed to AI assistants.
- **AI Integration:** Seamlessly integrates with AI assistants supporting the Model Context Protocol (MCP).

## Usage

The primary way to use this server is via `npx`, which downloads and runs the package without needing a local installation. This makes it easy to integrate with AI assistants and MCP clients (like IDE extensions).

### Installing via Smithery

To install Docs MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@buger/docs-mcp):

```bash
npx -y @smithery/cli install @buger/docs-mcp --client claude
```

### Integrating with MCP Clients (e.g., IDEs)

You can configure your MCP client to launch this server using `npx`. Here are examples of how you might configure a client (syntax may vary based on the specific client):

**Example 1: Dynamically Searching a Git Repository (Tyk Docs)**

This configuration tells the client to run the latest `@buger/docs-mcp` package using `npx`, pointing it dynamically to the Tyk documentation repository. The `-y` argument automatically confirms the `npx` installation prompt. The `--toolName` and `--toolDescription` arguments customize how the search tool appears to the AI assistant.

```json
{
  "mcpServers": {
    "tyk-docs-search": {
      "command": "npx",
      "args": [
        "-y",
        "@buger/docs-mcp@latest",
        "--gitUrl",
        "https://github.com/TykTechnologies/tyk-docs",
        "--toolName",
        "search_tyk_docs",
        "--toolDescription",
        "Search Tyk API Management Documentation"
      ],
      "enabled": true
    }
  }
}
```

Alternatively, some clients might allow specifying the full command directly. You could achieve the same as Example 1 using:

```bash
npx -y @buger/docs-mcp@latest --gitUrl https://github.com/TykTechnologies/tyk-docs --toolName search_tyk_docs --toolDescription "Search Tyk API Management Documentation"
```

**Example 2: Using a Pre-built, Branded MCP Server (e.g., Tyk Package)**

If a team publishes a pre-built package containing specific documentation (like `@tyk-technologies/docs-mcp`), the configuration becomes simpler as the content source and tool details are baked into that package. The `-y` argument is still recommended for `npx`.

```json
{
  "mcpServers": {
    "tyk-official-docs": {
      "command": "npx",
      "args": [
        "-y",
        "@tyk-technologies/docs-mcp@latest"
      ],
      "enabled": true
    }
  }
}
```

This approach is ideal for distributing standardized search experiences for official documentation or codebases. See the "Creating Your Own Pre-built MCP Server" section below.

Here is example on how Tyk team have build own documentation MCP server https://github.com/TykTechnologies/docs-mcp. 

## Configuration

Create a `docs-mcp.config.json` file in the root directory to define the **default** content source and MCP tool details used during the build and at runtime (unless overridden by CLI arguments or environment variables).

### Example 1: Using a Local Directory

```json
{
  "includeDir": "/Users/username/projects/my-project/docs",
  "toolName": "search_my_project_docs",
  "toolDescription": "Search the documentation for My Project.",
  "ignorePatterns": [
    "node_modules",
    ".git",
    "build",
    "*.log"
  ]
}
```

### Example 2: Using a Git Repository

```json
{
  "gitUrl": "https://github.com/your-org/your-codebase.git",
  "gitRef": "develop",
  "autoUpdateInterval": 15,
  "toolName": "search_codebase",
  "toolDescription": "Search the main company codebase.",
  "ignorePatterns": [
    "*.test.js",
    "dist/",
    "__snapshots__"
  ]
}
```

### Configuration Options

- `includeDir`: **(Build/Runtime)** Absolute path to a local directory whose contents will be copied to the `data` directory during build, or used directly at runtime if `dataDir` is not specified. Use this OR `gitUrl`.
- `gitUrl`: **(Build/Runtime)** URL of the Git repository. Use this OR `includeDir`.
    - If `autoUpdateInterval` is 0 (default), the server attempts to download a `.tar.gz` archive directly (currently assumes GitHub URL structure: `https://github.com/{owner}/{repo}/archive/{ref}.tar.gz`). This is faster but doesn't support updates.
    - If `autoUpdateInterval` > 0, the server performs a `git clone` and enables periodic updates.
- `gitRef`: **(Build/Runtime)** The branch, tag, or commit hash to use from the `gitUrl` (default: `main`). Used for both tarball download and Git clone/pull.
- `autoUpdateInterval`: **(Runtime)** Interval in minutes to automatically check for Git updates (default: 0, meaning disabled). Setting this to a value > 0 enables Git cloning and periodic `git pull` operations. Requires the `git` command to be available in the system path.
- `dataDir`: **(Runtime)** Path to the directory containing the content to be searched at runtime. Overrides content sourced from `includeDir` or `gitUrl` defined in the config file or built into the package. Useful for pointing the server to live data without rebuilding.
- `toolName`: **(Build/Runtime)** The name of the MCP tool exposed by the server (default: `search_docs`). Choose a descriptive name relevant to the content.
- `toolDescription`: **(Build/Runtime)** The description of the MCP tool shown to AI assistants (default: "Search documentation using the probe search engine.").
- `ignorePatterns`: **(Build/Runtime)** An array of glob patterns.
- `enableBuildCleanup`: **(Build)** If `true` (default), removes common binary/media files (images, videos, archives, etc.) and files larger than 100KB from the `data` directory after the build step. Set to `false` to disable this cleanup.
    - If using `includeDir` during build: Files matching these patterns are excluded when copying to `data`. `.gitignore` rules are also respected.
    - If using `gitUrl` or `dataDir` at runtime: Files matching these patterns within the `data` directory are ignored by the search indexer.

**Precedence:**

1.  **Runtime Configuration (Highest):** CLI arguments (`--dataDir`, `--gitUrl`, etc.) and Environment Variables (`DATA_DIR`, `GIT_URL`, etc.) override all other settings. CLI arguments take precedence over Environment Variables.
2.  **Build-time Configuration:** Settings in `docs-mcp.config.json` (`includeDir`, `gitUrl`, `toolName`, etc.) define defaults used during `npm run build` and also serve as runtime defaults if not overridden.
3.  **Default Values (Lowest):** Internal defaults are used if no configuration is provided (e.g., `toolName: 'search_docs'`, `autoUpdateInterval: 5`).

Note: If both `includeDir` and `gitUrl` are provided in the *same* configuration source (e.g., both in the config file, or both as CLI args), `gitUrl` takes precedence.

## Creating Your Own Pre-built MCP Server

You can use this project as a template to create and publish your own npm package with documentation or code pre-built into it. This provides a zero-configuration experience for users (like Example 2 above).

1.  **Fork/Clone this Repository:** Start with this project's code.
2.  **Configure `docs-mcp.config.json`:** Define the `includeDir` or `gitUrl` pointing to your content source. Set the default `toolName` and `toolDescription`.
3.  **Update `package.json`:** Change the `name` (e.g., `@my-org/my-docs-mcp`), `version`, `description`, etc.
4.  **Build:** Run `npm run build`. This clones/copies your content into the `data` directory and makes the package ready.
5.  **Publish:** Run `npm publish` (you'll need npm authentication configured).

Now, users can run your specific documentation server easily: `npx @my-org/my-docs-mcp@latest`.

*(The previous "Running", "Dynamic Configuration at Runtime", and "Environment Variables" sections have been removed as `npx` usage with arguments within client configurations is now the primary documented method.)*

## Using with AI Assistants

This MCP server exposes a search tool to connected AI assistants via the Model Context Protocol. The tool's name and description are configurable (see Configuration section). It searches the content within the currently active `data` directory (determined by build settings, config file, CLI args, or environment variables).

**Tool Parameters:**

- `query`: A natural language query or keywords describing what to search for (e.g., "how to configure the gateway", "database connection example", "user authentication"). The server uses Probe's search capabilities to find relevant content. (Required)
- `page`: The page number for results when dealing with many matches. Defaults to 1 if omitted. (Optional)

**Example Tool Call (using `search_tyk_docs` from Usage Example 1):**

```json
{
  "tool_name": "search_tyk_docs",
  "arguments": {
    "query": "gateway rate limiting",
    "page": 1 // Requesting the first page
  }
}
```

**Example Tool Call (using the tool from the `@tyk/docs-mcp` package):**

Assuming the pre-built package `@tyk/docs-mcp` defined its tool name as `search_tyk_official_docs`:

```json
{
  "tool_name": "search_tyk_official_docs",
  "arguments": {
    "query": "dashboard api access",
    "page": 2 // Requesting the second page
  }
}
```

*(The previous "Publishing as an npm Package" section has been replaced by the "Creating Your Own Pre-built MCP Server" section above.)*

## License

MIT
