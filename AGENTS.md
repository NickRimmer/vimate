# Repository Guidelines

## Project Structure & Module Organization
- `src/ViMate/ViMate.slnx`: solution entry point.
- `src/ViMate/ViMate.Framework/`: core framework library (`ViMate.Framework.csproj`).
- `src/ViMate/Samples/ViMate.Samples.Console/`: runnable sample app for validating framework behavior.
- `docs/concept.md`: product and architecture concept notes for v0.

Keep reusable runtime logic in `ViMate.Framework`; keep demos, experiments, and manual verification flows in `Samples`.

## Build, Test, and Development Commands
Run commands from repository root unless noted.
- `dotnet restore src/ViMate/ViMate.slnx`: restore NuGet dependencies.
- `dotnet build src/ViMate/ViMate.slnx -c Debug`: build all projects.
- `dotnet run --project src/ViMate/Samples/ViMate.Samples.Console`: run the sample console app.
- `dotnet test src/ViMate/ViMate.slnx`: execute tests when test projects exist.

Use `-c Release` for production-like validation.

## Coding Style & Naming Conventions
- C# style follows SDK defaults: 4-space indentation, UTF-8 files, nullable reference types enabled.
- Use `PascalCase` for public types/members, `camelCase` for locals/parameters, and descriptive names (e.g., `ContextInjectionOrchestrator`).
- Keep one public type per file and match file names to type names.
- Prefer small, composable classes in framework code; keep sample `Program.cs` focused on usage examples.

## Testing Guidelines
- Preferred test stack: xUnit with `dotnet test`.
- Place tests under `src/ViMate/tests/` (example: `src/ViMate/tests/ViMate.Framework.Tests/`).
- Name test files as `<TypeName>Tests.cs` and test methods as `MethodName_State_ExpectedResult`.
- Prioritize unit tests for scheduling, context priority ordering, and tool orchestration behavior.

## Commit & Pull Request Guidelines
- Current history uses imperative, concise subjects (example: `Add initial project structure ...`).
- Use commit format: `Verb + scope + intent` (example: `Add context bucket priority ordering`).
- PRs should include: purpose summary, key design decisions, test evidence (`dotnet build`/`dotnet test` output), and linked issue(s).
- For sample behavior changes, include a short run transcript or screenshot of console output.

## Security & Configuration Tips
- Do not commit secrets or local machine paths.
- Keep environment-specific configuration out of source; prefer local overrides and documented defaults.
