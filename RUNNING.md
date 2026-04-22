# RUNNING

## Environment Preparation

Install Docker with the `docker compose` plugin, jq, and just before
interacting with this repository:

```bash
# macOS (Homebrew)
brew install docker jq just

# Ubuntu / Debian
sudo apt-get update
sudo apt-get install -y docker.io jq just

docker --version
docker compose version
jq --version
just --version
```

## Environment variables

Create a project-local environment file to control libFuzzer flags:

```bash
cp .env.example .env
# edit LIBFUZZ_* entries as needed
```

Consult the upstream
[libFuzzer](https://llvm.org/docs/LibFuzzer.html#options) docs to better
understand the options:

## Run Options

### Option 1 - Quick run via `just`

```bash
just run <fuzz target>

# e.g.
just run script
just run descriptor_parse
```

> [!NOTE]
> Each target relies on a writable `/app/data`. The default compose
> configuration binds `./docker` on the host. Ensure it exists (`mkdir -p
> docker`) or adjust the volume mapping before running.

## First Fuzzing Session

If you are new to libFuzzer output, start with the `script` target because it
has a small, easy-to-explain module set in `docker-compose.yml`:

- `BITCOIN_CORE`
- `RUST_BITCOIN`

That makes it a good first end-to-end run.

### 1. Prepare a short, bounded run

```bash
cp .env.example .env
mkdir -p docker
```

For a quick smoke test, set a low run count in `.env`:

```bash
LIBFUZZ_RUNS=1000
```

### 2. Start the target

Use either the convenience wrapper:

```bash
just run script
```

or the equivalent Docker Compose command directly:

```bash
docker compose up script --build --force-recreate
```

### 3. What you are looking at

You will typically see libFuzzer lines in this shape:

```text
INFO: Running with entropic power schedule (0xFF, 100).
INFO: Seed: 123456789
INFO: A corpus is not provided, starting from an empty corpus
#2 INITED cov: 110 ft: 126 corp: 1/1b exec/s: 0 rss: 45Mb
#17 NEW    cov: 128 ft: 149 corp: 2/14b lim: 4 exec/s: 0 rss: 46Mb L: 13/13 MS: 5 InsertByte-ChangeBit-...
#1000 DONE cov: 141 ft: 167 corp: 6/123b lim: 16 exec/s: 250 rss: 47Mb
```

How to read those lines:

- `INFO: Running with entropic power schedule`: libFuzzer is telling you which
  mutation scheduling strategy it will use.
- `INFO: Seed: ...`: the PRNG seed for this run. Save it if you need to replay
  a specific behavior later.
- `starting from an empty corpus`: there was no existing seed corpus in the
  mounted data directory, so libFuzzer is inventing inputs from scratch.
- `#2 INITED`: the engine has initialized the corpus for this target.
- `cov`: edge coverage reached so far. Higher means the target is exploring new
  code paths.
- `ft`: feature count tracked by libFuzzer. This is often more sensitive than
  raw coverage when you compare progress.
- `corp`: current corpus count and total corpus size in bytes.
- `lim`: the current maximum input size libFuzzer is trying.
- `exec/s`: executions per second.
- `rss`: resident memory use.
- `L: 13/13`: input length for the current interesting sample and the maximum
  length seen in this batch.
- `MS: ...`: the mutation sequence that produced the new interesting input.
- `DONE`: the configured run bound was reached cleanly.

If the run keeps printing `NEW`, that is usually a good sign: the target is
still discovering interesting inputs and adding them to the corpus.

### 4. Where the artifacts go

The compose file binds `./docker` on the host to `/app/data` in the container.
For the `script` service that means:

- corpus files land in `docker/script/corpus/`
- crash files land in `docker/script/crash/`
- merge bookkeeping goes to `docker/script/merge`

Inspect them from the host after the container exits:

```bash
ls -lah docker/script
ls -lah docker/script/corpus
ls -lah docker/script/crash
```

An empty `docker/script/crash/` directory after a short run is normal. It just
means the target did not trigger a sanitizer or assertion failure in that
session.

### 5. How to inspect a crash artifact

When libFuzzer finds a reproducer, it writes a file under
`docker/<target>/crash/`, usually with a name like `crash-*` or `timeout-*`.

Start by identifying the newest file:

```bash
ls -lt docker/script/crash
```

Then rerun the same target against only that artifact:

```bash
LIBFUZZ_RUNS=1 docker run --rm \
  -e LIBFUZZ_JOBS=1 \
  -e LIBFUZZ_FORKS=1 \
  --env-file .env \
  -v "$(pwd)/docker":/app/data \
  bitcoinfuzz:script \
  /app/data/script/crash/crash-1234567890abcdef
```

The explicit `LIBFUZZ_RUNS=1`, `LIBFUZZ_JOBS=1`, and `LIBFUZZ_FORKS=1` overrides
keep this in reproduction mode instead of resuming a longer fuzzing session from
the values stored in `.env`.

If you only want to look at the raw bytes first:

```bash
hexdump -C docker/script/crash/crash-1234567890abcdef | head
```

That workflow gives you the three core pieces you need for triage:

- the exact reproducer input
- the target that crashed
- the saved corpus/crash state under `docker/`

### Option 2 - Custom run via Docker Compose

1. Modify `docker-compose.yml` (build args, env vars, new services, cpu quota,
   etc.).
2. Launch services:

```bash
docker compose up <fuzz target | compose service> --build --force-recreate

# e.g.
docker compose up script --build --force-recreate
docker compose up descriptor_parse miniscript_parse --build

# later, to stop and remove containers
docker compose down
```

> [!NOTE]
> Keep the `./docker:/app/data` bind (or swap for a named volume like `docker
> volume create bitcoinfuzz-data`) so crashes/corpora persist on the host.

### Option 3 - Manual `docker run`

```bash
mkdir docker
# See auto_build.py and Dockerfile to understand better the build args
docker build -t bitcoinfuzz:script \
  --build-arg FUZZ=script \
  --build-arg "CXXFLAGS=-DBITCOIN_CORE -DRUST_BITCOIN" .

docker run --rm \
  --name bitcoinfuzz-script \
  -e FUZZ=script \
  -e LIBFUZZ_TIMEOUT=300 \
  -v "$(pwd)/docker":/app/data \
  bitcoinfuzz:script

# You can also limit resources to the container
# Use the .env file, bind a writable data volume, and limit host resources.
docker run --rm \
  --name bitcoinfuzz-transaction_eval \
  --env-file .env \
  -v "$(pwd)/docker":/app/data \
  --cpus 2 \
  --memory 2g \
  --net none \
  bitcoinfuzz:transaction_eval
```

> [!NOTE]
> Replace the bind mount with a named volume if preferred: `docker run ... -v
> bitcoinfuzz-data:/app/data ...`. Be aware that folder will be created.

### Selective Module Loading

You can use the `MODULES` environment variable to load only specific modules at
runtime, without rebuilding the image:

```bash
# Load only Bitcoin Core and rust-bitcoin
docker run --rm \
  -e MODULES="BITCOIN_CORE,RUST_BITCOIN" \
  -v "$(pwd)/docker":/app/data \
  bitcoinfuzz:script

# Load only Lightning implementations
docker run --rm \
  -e MODULES="LDK,LND,CLIGHTNING" \
  -v "$(pwd)/docker":/app/data \
  bitcoinfuzz:deserialize_invoice

# With docker compose
MODULES="BITCOIN_CORE,RUST_BITCOIN" docker compose up deserialize_block
```

### Option 4 - Running from Source

To build outside of Docker (useful for local debugging), execute `auto_build.py`
with the required flags:

```bash
# pass in the modules you want to compile with -D<mod> in CXXFLAGS
CXXFLAGS="-DBITCOIN_CORE -DRUST_BITCOIN" ./auto_build.py
```

This script cleans and builds modules based on `CXXFLAGS`, and then compiles
the root project.
