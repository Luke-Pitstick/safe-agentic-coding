# Ignore Pattern Guidance

Use these patterns as candidates, not as a blanket template. Add only what matches the repo's actual files and tooling.

## Always Common

- OS/editor noise: `.DS_Store`, `Thumbs.db`, `.idea/`, `.vscode/` only when workspace settings are not intentionally shared.
- Logs/temp: `*.log`, `*.tmp`, `*.temp`, `tmp/`, `temp/`.
- Local env/secrets: `.env`, `.env.*`, `!.env.example`, `!.env.sample`, `*.pem`, `*.key`, `*.p12`, `*.pfx` when not intentional fixtures.
- Coverage/test output: `coverage/`, `.coverage`, `htmlcov/`, `test-results/`, `playwright-report/`.

## JavaScript and TypeScript

- Dependencies: `node_modules/`.
- Builds/caches: `dist/`, `build/`, `.next/`, `.nuxt/`, `.svelte-kit/`, `.astro/`, `.vite/`, `.turbo/`, `.parcel-cache/`, `.cache/`.
- Package manager metadata usually stays tracked: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lock`, `bun.lockb`.

## Python

- Bytecode/cache: `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, `.tox/`, `.nox/`.
- Virtual envs: `.venv/`, `venv/`, `env/`.
- Build artifacts: `build/`, `dist/`, `*.egg-info/`.
- Usually keep `pyproject.toml`, `uv.lock`, `poetry.lock`, and requirements files tracked.

## Rust, Go, Java, .NET, Ruby, PHP

- Rust: `target/`; keep `Cargo.lock` for applications, ask before ignoring it for libraries.
- Go: avoid broad build ignores unless artifacts are present; keep `go.sum`.
- Java/Kotlin: `target/`, `build/`, `.gradle/`, `out/`; keep Gradle/Maven config and wrapper files.
- .NET: `bin/`, `obj/`, `TestResults/`; keep project and solution files.
- Ruby: `.bundle/`, `vendor/bundle/`, `log/`, `tmp/`; keep `Gemfile.lock` for apps.
- PHP: `vendor/` if dependencies are installed by Composer; keep `composer.lock` for apps.

## Mobile and Desktop

- Xcode/Swift: `.build/`, `DerivedData/`, `*.xcuserstate`, `xcuserdata/`; do not ignore project files by default.
- Android: `.gradle/`, `build/`, `local.properties`; keep `gradle-wrapper.jar` and wrapper scripts.
- Flutter/Dart: `.dart_tool/`, `build/`; keep `pubspec.lock` for apps, ask for libraries.

## Keep Tracked Unless Clearly Generated

- Source code, tests, migrations, schemas, docs, and design assets.
- Lockfiles for applications.
- CI config, deployment config, Dockerfiles, compose files.
- Example env files and fixture data intentionally used by tests.
- Generated code that the repo conventionally commits, such as API clients or protobuf output, unless the build process clearly regenerates it and the team wants it removed.
