# Svelte 5 & SvelteKit Skill for Claude Code

A comprehensive Claude Code skill for Svelte 5 and SvelteKit development.

## Features

- **Svelte 5 Runes**: `$state`, `$derived`, `$effect`, `$props`, `$bindable`, `$inspect`
- **Rune Variants**: `.raw`, `.snapshot`, `.by`, `.pre`, `.tracking`, `.root`
- **Snippets & Rendering**: Modern slot replacement with `{#snippet}` and `{@render}`
- **SvelteKit**: Routing, data loading, form actions, API endpoints, hooks
- **TypeScript**: Full type safety patterns
- **Security**: Best practices for secure Svelte applications

## Installation

Add the marketplace:

```
/plugin marketplace add xeros0201/svelte5_skills
```

Install the plugin:

```
/plugin install svelte-skill@svelte5-skills
```

## Usage

The skill automatically activates when working with Svelte or SvelteKit code. Claude will use the documented patterns for:

- Creating Svelte 5 components with runes
- Setting up SvelteKit routes and layouts
- Implementing form actions and API endpoints
- Managing state with `$state` and `$derived`
- Handling authentication and authorization
- Following security best practices

## Documentation Sources

This skill references official Svelte documentation:

- https://svelte.dev/docs/svelte/llms.txt
- https://svelte.dev/docs/kit/llms.txt

## License

MIT
