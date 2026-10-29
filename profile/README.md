# Prismalyst

Static analysis for Prisma queries in TypeScript.

Prismalyst helps you catch risky database operations and common query-performance mistakes while writing code and running ESLint in CI. It uses TypeScript type information to recognize Prisma calls, then checks their arguments, surrounding code, and, where needed, your Prisma schema.

**0.1.0 is an early beta for Prisma 7**, with 16 rules, including four that use schema metadata. Analysis does not execute queries or require a database connection.

## What it checks

| Area | Examples |
| --- | --- |
| Bulk operations | Missing, undefined, or empty `where` filters in `deleteMany` and `updateMany`. |
| Raw SQL | Unsafe raw-query APIs, interpolated SQL templates, and dynamic values passed to `Prisma.raw()`. |
| Query size and pagination | Missing or excessive `take`, deep offset pagination, and ordering without a unique tie-breaker. |
| Query execution | Unhandled Prisma promises and queries inside loops. |
| Client lifecycle and transactions | Client construction inside functions, disconnection outside allowed files, and root-client use inside interactive transactions. |
| Schema-aware checks | Unindexed filter keys, cascading deletes, unstable pagination, and unbounded list-relation loading. |

For example, with a configured client and a `User` model:

```ts
// Reported by require-where-delete-many: no filter.
await prisma.user.deleteMany();

// Reported by max-take: above its default maximum of 100.
await prisma.user.findMany({ take: 500 });
```

Depending on the operation you intended, you could narrow the deletion or bound the query:

```ts
await prisma.user.deleteMany({ where: { id: 42 } });

await prisma.user.findMany({
  take: 50,
  orderBy: { id: 'asc' },
});
```

These examples illustrate the named checks. Other enabled rules may also report on a query, depending on your schema and configuration.

## Get started

In an existing Prisma 7 project, install the ESLint plugin and its linting tools:

```sh
npm install --save-dev @prismalyst/eslint-plugin eslint@^10 typescript@~6.0.3 typescript-eslint@^8
```

Then follow the [plugin quick start](https://github.com/Prismalyst/eslint-plugin#quick-start) to configure ESLint with TypeScript type information. The plugin installs the core engine automatically.

- [Configure options and severity, disable rules, or exclude files](https://github.com/Prismalyst/eslint-plugin#configure-rules)
- [Browse all 16 rules and their defaults](https://github.com/Prismalyst/eslint-plugin#rules-and-defaults)
- [Understand schema discovery](https://github.com/Prismalyst/eslint-plugin#schema-discovery)
- [Read detailed rule documentation](https://github.com/Prismalyst/core#rules)

The four schema-dependent rules need a discoverable `schema.prisma`. The other rules work without schema discovery. All rules require TypeScript type information and a generated client that the engine can recognize.

## Repositories

Prismalyst is developed across separate repositories. This repository provides the product overview; package setup, development commands, and implementation details live with each component.

| Repository | Package | Responsibility |
| --- | --- | --- |
| [ESLint plugin](https://github.com/Prismalyst/eslint-plugin) | `@prismalyst/eslint-plugin` | ESLint integration, configuration, rule messages, options, and schema loading. Start here to use Prismalyst. |
| [Core](https://github.com/Prismalyst/core) | `@prismalyst/core` | Prisma expression detection, argument parsing, rule implementations, and shared TypeScript contracts. |

## Current compatibility

| Tool | 0.1.0 scope |
| --- | --- |
| Prisma | Prisma 7. |
| TypeScript | TypeScript 6.0 is the development baseline. The declared peer range is `>=5.0.0 <7`; not every version in that range has been verified. |
| ESLint | ESLint 10 with flat configuration and TypeScript type information. |
| Node.js | Node.js 24 is the development and CI baseline. |

## Beta limitations

Reports are checks to review, rather than proof of query safety or database performance. Some rules can produce false positives or miss cases, particularly promise handling and index analysis. Client recognition also depends on generated-file paths, and schema discovery currently selects a single `schema.prisma` automatically.

Review the [plugin's known limitations](https://github.com/Prismalyst/eslint-plugin#known-limitations) and the documentation for the rules you enable. In particular, the current `all` preset enables the approximate promise rule as an error; you can disable individual rules or change their severity before using the preset in CI.

## Roadmap

- Add Prisma 8 support after its stable release.
- Add TypeScript 7 support targeting the stable TypeScript 7.1 compiler API.

These are planned compatibility additions, outside the 0.1.0 support scope.

## Contributing and feedback

Use the relevant component repository for changes and bug reports:

- [Plugin issues](https://github.com/Prismalyst/eslint-plugin/issues): configuration, schema loading, and ESLint integration.
- [Core issues](https://github.com/Prismalyst/core/issues): query detection, parsed arguments, and rule behavior.

For a rule report, include a small code example, expected and actual behavior, relevant schema models, configuration, and tool versions. Follow the development instructions in the component's README before submitting a change.

## License

Both the [core](https://github.com/Prismalyst/core/blob/main/LICENSE) and [ESLint plugin](https://github.com/Prismalyst/eslint-plugin/blob/main/LICENSE) are MIT licensed.
