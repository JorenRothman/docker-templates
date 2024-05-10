# Caddy

## Pre Build Compressed Files

It's advised to pre compile your static files to both brotli and gzip.

### Astro example

```bash
pnpm astro add astro-compressor
```

```astro.config.mjs
import compressor from 'astro-compressor';

export default defineConfig({
    integrations: [
        ...
        compressor(),
    ]
});
```
