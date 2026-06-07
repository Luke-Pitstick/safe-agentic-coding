# Framework Commands

Use this reference to identify likely test commands and file conventions. Prefer commands already defined in project scripts, Makefiles, task runners, CI files, or docs.

## Discovery

Look for:

- `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `vitest.config.*`, `jest.config.*`, `playwright.config.*`, `cypress.config.*`
- `pyproject.toml`, `pytest.ini`, `tox.ini`, `noxfile.py`, `setup.cfg`
- `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `build.gradle.kts`
- `Package.swift`, `.xcodeproj`, `.xcworkspace`
- `.github/workflows/*`, `Makefile`, `justfile`, `Taskfile.yml`

Useful searches:

```bash
rg --files | rg '(^|/)(test|tests|spec|__tests__|e2e)(/|$)|(_test|\\.test|\\.spec)\\.'
rg -n '"test"|pytest|vitest|jest|playwright|cargo test|go test|swift test|mvn test|gradle test'
```

## JavaScript and TypeScript

Prefer package scripts:

```bash
npm test
npm run test
pnpm test
bun test
yarn test
```

Common targeted commands:

```bash
npx vitest run path/to/file.test.ts
npx jest path/to/file.test.ts
npx playwright test path/to/spec.ts
npx cypress run --spec path/to/spec.cy.ts
```

File conventions: `*.test.ts`, `*.spec.ts`, `__tests__/`, `test/`, `tests/`, `e2e/`.

## Python

Prefer project runner:

```bash
uv run pytest
pytest
python -m pytest
tox
nox
```

Targeted:

```bash
uv run pytest path/to/test_file.py::test_name
python -m unittest path.to.test_module
```

File conventions: `test_*.py`, `*_test.py`, `tests/`.

## Go

```bash
go test ./...
go test ./path/to/package
go test ./path/to/package -run TestName
```

File convention: `*_test.go`.

## Rust

```bash
cargo test
cargo test -p package_name
cargo test test_name
```

File conventions: inline `#[cfg(test)]`, `tests/*.rs`.

## Java and Kotlin

Maven:

```bash
mvn test
mvn -Dtest=ClassName test
```

Gradle:

```bash
./gradlew test
./gradlew test --tests 'package.ClassName.testName'
```

File conventions: `src/test/java`, `src/test/kotlin`.

## Swift

```bash
swift test
swift test --filter TestName
xcodebuild test -scheme SchemeName
```

File conventions: `Tests/`, `*Tests.swift`.

## Ruby

```bash
bundle exec rspec
bundle exec rspec path/to/spec.rb
bin/rails test
```

File conventions: `spec/`, `test/`, `*_spec.rb`.

## .NET

```bash
dotnet test
dotnet test --filter FullyQualifiedName~TestName
```

File conventions: `*.Tests`, `*Tests.cs`.
