Short URL service deployed on Cloudflare Workers

Using Cloudflare's Workers routing configuration, you can route `/` to GitHub Pages and `/*` to Cloudflare Workers, so that the homepage will not occupy Workers resources
## frontend

Standard Vue project, nothing to say

Attached GitHub Actions are automatically deployed to the gh-pages branch, but you may need to enable GitHub Pages in Settings for the first time
### Secrets

-`ADDITION_HEAD`-will be added before `</head>`, where statistics codes, etc. can be placed
## backend

Use the free KV provided by Cloudflare Workers to store. The original URL is up to 1024 in length, and the generated short URL ID is Base56 with a length of 6 `23456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz`

Hash algorithm is preferred to generate ID. Each URL can generate 4 IDs through hash algorithm. If all are occupied, randomly generated IDs will be used, with a maximum of 1000 attempts

### KV

Create a new namespace and bind it to the `URL_DB` variable
