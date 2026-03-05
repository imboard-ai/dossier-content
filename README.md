# Dossier Content

The content repository for all published [Dossier](https://github.com/imboard-ai/ai-dossier) automation instructions.

This repo is managed by the [Dossier Registry](https://github.com/imboard-ai/dossier-registry) -- dossiers are published via the CLI and committed here automatically. Content is served to users via jsDelivr CDN.

## Structure

```
index.json              # Manifest of all published dossiers
getting-started/        # Tutorial dossiers
imboard-ai/             # ImBoard.ai namespace
  skills/               # Reusable AI agent skills
  development/          # Development workflow dossiers
  meta/                 # Dossier authoring guides
tal-liberio/            # Tal-Liberio namespace
  setup/                # Setup validation dossiers
  development/          # Development validation dossiers
```

## Publishing

Dossiers are published through the CLI, not by committing directly to this repo:

```bash
ai-dossier login
ai-dossier publish ./my-dossier.ds.md --namespace my-namespace/category
```

## Related Projects

| Project | Description |
|---------|-------------|
| [ai-dossier](https://github.com/imboard-ai/ai-dossier) | Core library, CLI, and MCP server |
| [dossier-registry](https://github.com/imboard-ai/dossier-registry) | Registry API for publishing and discovery |

## License

This project is licensed under the [Elastic License 2.0 (ELv2)](LICENSE). You are free to use, copy, modify, and distribute it, with one restriction: you may not offer it as a hosted or managed service to third parties.
