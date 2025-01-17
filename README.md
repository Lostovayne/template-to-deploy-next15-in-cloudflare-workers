# Next.js 15 on Cloudflare Workers - Implementation Template

This template demonstrates how to implement Next.js 15.1.4 on Cloudflare Workers using @opennextjs/cloudflare. This README provides a step-by-step guide to configure and deploy your application.

## Dependency Versions

### Main Dependencies

- next: ^15.1.4
- react: ^19.0.0
- react-dom: ^19.0.0
- @opennextjs/cloudflare: ^0.3.8
- wrangler: ^3.103.1

### Development Dependencies

- @cloudflare/workers-types: ^4.20250109.0
- typescript: ^5
- tailwindcss: ^3.4.1
- postcss: ^8
- eslint: ^9
- eslint-config-next: 15.1.4

## Step-by-Step Implementation Guide

### 1. Initial Setup

```bash
# Create new project (Option 1)
npm create cloudflare@latest -- my-next-app --framework=next --experimental

# Or configure existing project (Option 2)
bun add  @opennextjs/cloudflare@latest
bun add  wrangler@latest
bun add -D @cloudflare/workers-types
```

### 2. File Configuration

#### 2.1 Create wrangler.json

```json
{
	"main": ".open-next/worker.js",
	"name": "my-app",
	"compatibility_date": "2024-12-30",
	"compatibility_flags": ["nodejs_compat"],
	"assets": {
		"directory": ".open-next/assets",
		"binding": "ASSETS"
	}
}
```

#### 2.2 Create open-next.config.ts

```typescript
import type { OpenNextConfig } from "@opennextjs/aws/types/open-next.js";
import cache from "@opennextjs/cloudflare/kvCache";

const config: OpenNextConfig = {
	default: {
		override: {
			wrapper: "cloudflare-node",
			converter: "edge",
			incrementalCache: async () => cache,
			tagCache: "dummy",
			queue: "dummy"
		}
	},
	middleware: {
		external: true,
		override: {
			wrapper: "cloudflare-edge",
			converter: "edge",
			proxyExternalRequest: "fetch"
		}
	}
};

export default config;
```

#### 2.3 Environment Variables Setup

1. Create `.dev.vars` file:

```plaintext
NEXTJS_ENV=development
```

2. (Optional) Create `.dev.vars.example` as a template for other developers.

### 3. Package.json Configuration

Add the following scripts:

```json
{
	"scripts": {
		"dev": "next dev --turbopack",
		"build": "next build",
		"start": "next start",
		"lint": "next lint",
		"build:worker": "opennextjs-cloudflare",
		"dev:worker": "wrangler dev --port 8771",
		"preview": "npm run build:worker && npm run dev:worker",
		"deploy": "npm run build:worker && wrangler deploy",
		"cf-typegen": "wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts"
	}
}
```

## Available Commands

- `npm run dev`: Start development server with Turbopack
- `npm run build`: Build Next.js application
- `npm run build:worker`: Build Cloudflare worker
- `npm run dev:worker`: Start Wrangler development server
- `npm run preview`: Build and preview worker locally
- `npm run deploy`: Build and deploy worker to Cloudflare
- `npm run cf-typegen`: Generate TypeScript types for Cloudflare environment

## Important Notes

1. Make sure you have a Cloudflare account and are authenticated with Wrangler.
2. Minimum required Wrangler version is 3.99.0.
3. The `.dev.vars` file should not be shared or committed to the repository.
4. To enable KV cache, uncomment and configure the `kv_namespaces` section in `wrangler.json`.

## Project Structure

```
├── .dev.vars                 # Development environment variables
├── .dev.vars.example         # Environment variables template
├── open-next.config.ts       # OpenNext configuration
├── wrangler.json            # Cloudflare Workers configuration
├── package.json             # Dependencies and scripts
└── README.md               # This file
```

## Contributing

If you find any issues or have suggestions for improvements, please feel free to open an issue or submit a pull request.
