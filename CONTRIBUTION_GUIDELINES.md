# Contribution Guidelines

If you ever feel like wanting to help out with anything, please do! Any help at all is very, very much appreciated.
Refactors, cleanups, new features, bug fixes - We'll take it all!

A good place to start right now is by looking at the Chorus or Symfonia repositories and their roadmaps.
If you're new to Rust or coding in general, feel free to check out the issues labeled as 'Difficulty: Simple' or 'Difficulty: Trivial'.

If you want to contribute code, please follow these steps:

- Find a feature or issue you want to work on. If you want to work on something that is not listed, open an issue and we will discuss it.
  - Minor bug fixes as well as minor refactors do not necessarily need to be an issue. Anything more major than that, like feature additions, do.
- **Let it be known in the issue that you are working on it.** This is to avoid duplicate work, especially since currently this project is still in its early stages and there are not many contributors.
- Fork the Project/Create a feature branch
- Create your Feature Branch (git checkout -b feature/AmazingFeature)
- Commit your changes (git commit -m 'Add some AmazingFeature')
- Push to the Branch (git push origin feature/AmazingFeature)
- Open a Pull Request
  - If applicable, open the PR for the `dev` branch instead of the `main` branch.
  - Be descriptive in the title and description of your PR.
  - The description of your PR should at least outline the work you have done.

## Important Places to Contribute to

If you're not fluent in Rust:

- Write documentation where required and fix typos in existing documentation (Search for `DOCUMENTME` in the code).
- Use/test our software, file issues for bugs you find, and suggest features you would like to see.
- Web development (Polyphony Web/Tauri-repo)
- Write unit tests for existing code, if you're feeling adventurous.

If you are fluent in Rust:

- Search for `TODO, FIXME, BUG, UNOPTIMIZED, REWRITEME` and `PRETTYFYME` in the codebase and fix the issues they point out.
- Remove clippy warnings by fixing the issues they point out.
- Check the issues labeled as 'Difficulty: Simple' or 'Difficulty: Trivial' and fix them.
- Ask a maintainers for a task to work on.

## Versioning

All projects in this organization use [Semantic Versioning](https://semver.org/). This means that the version number is incremented as follows: `MAJOR.MINOR.PATCH`, where `MAJOR` is incremented when incompatible API changes are made, `MINOR` is incremented when new functionality is added in a backward-compatible manner, and `PATCH` is incremented when backward-compatible bug fixes are made.
