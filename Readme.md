# jurriaanr/frankenphp-worker

[FrankenPHP](https://frankenphp.dev) in **worker mode** — the PHP application boots once and stays in memory, handling requests in a persistent event loop. Typically 3-5× faster than per-request PHP execution.

Based on the official `dunglas/frankenphp` image — see `Dockerfile` for the pinned version.

For the standard per-request image (safer default, compatible with any PHP app), see `jurriaanr/frankenphp`.

## Requirements

**Your application must be written for worker mode.** Globals, static properties, and in-memory caches persist between requests. Running a vanilla PHP script (with top-level `echo` statements or a traditional include-based structure) will misbehave or outright fail.

Known compatible runtimes:

- **Symfony 7.4+** — `symfony/runtime` supports FrankenPHP worker mode natively; no extra package needed
- **Laravel Octane** (separate configuration)
- Custom code using `frankenphp_handle_request()` in a `while` loop

If your app isn't one of these, use `jurriaanr/frankenphp` instead.

## What's included

Identical to `jurriaanr/frankenphp`:

- Same PHP extensions: `bcmath, curl, exif, gd, intl, imagick, ldap, mbstring, memcached, mysqli, opcache, pdo_mysql, pdo_pgsql, simplexml, soap, sockets, redis, xmlrpc, xsl, zip`
- Same locales, timezone, security headers, compression, caching
- Composer 2, git, ssh-client, cron

The only difference is the Caddyfile: its global `frankenphp` block declares a `worker` on `public/index.php` and sets `FRANKENPHP_WORKER=1` in the worker's environment, which is what `symfony/runtime` checks to switch to its worker runner.

## Environment variables

Same as `jurriaanr/frankenphp`, plus:

| Variable | Default | Description |
|---|---|---|
| `FRANKENPHP_WORKER_FILE` | `/app/public/index.php` | Worker script path. Override to point at a different entry. |
| `FRANKENPHP_NUM_WORKERS` | `2` | Number of worker threads. |
| `FRANKENPHP_CONFIG` | _(empty)_ | Extra directives inserted into the global `frankenphp { }` block (e.g. `num_threads 8`). |

## Usage

For a Symfony 7.4+ app:

```yaml
services:
  app:
    image: jurriaanr/frankenphp-worker:latest
    ports:
      - "80:80"
    volumes:
      - .:/app
    environment:
      FRANKENPHP_NUM_WORKERS: 4
```

No `APP_RUNTIME` or extra package is needed: the stock `public/index.php` from Symfony Flex works as the worker script.

## Trade-offs

**Pros**

- Significantly faster per request (no bootstrap overhead)
- Container warm start is fast (Symfony kernel already booted)
- Opcache and in-memory caches stay warm between requests

**Cons**

- Requires a worker-compatible application
- Memory leaks accumulate across requests; the worker restarts periodically to mitigate
- Debugging is trickier (state persists, xdebug integrates differently)
- Static analysis tools may flag patterns that are fine in per-request mode but problematic here

**When to choose standard mode instead**

- Running vanilla PHP code (WordPress, legacy apps, custom scripts)
- Development with frequent code changes (worker needs restarts to pick up some changes)
- When the performance gap isn't worth the compatibility investigation

## Building

```bash
docker image build --network=host -t frankenphp-worker .
docker image tag frankenphp-worker jurriaanr/frankenphp-worker:latest
docker push jurriaanr/frankenphp-worker:latest
```

Or use the `Update-DockerImages.ps1` script:

```bash
./Update-DockerImages.ps1 -Tool frankenphp-worker
```

## Testing locally

Requires a Symfony 7.4+ app. A vanilla `index.php` won't work — that's by design for worker mode.

Basic smoke test against a Symfony app:

```bash
curl -I http://localhost/                    # security headers, 200 OK
curl    http://localhost/                    # Symfony renders

# Confirm worker mode: SERVER_SOFTWARE should include frankenphp
curl http://localhost/_profiler              # Symfony profiler shows request time
```

Worker mode dramatically reduces "bootstrap" time shown in the Symfony profiler — if you see numbers similar to standard mode, the worker runner isn't active (check that `symfony/runtime` is 7.4+).

## See also

- [FrankenPHP worker mode docs](https://frankenphp.dev/docs/worker/)
- [Symfony Runtime component](https://symfony.com/doc/current/components/runtime.html)