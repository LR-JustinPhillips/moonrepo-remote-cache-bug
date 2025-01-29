# moonrepo-remote-cache-bug

## The bug:

When rehydrating a dependency, files some files are not restored. Specifically, if there are multiple files with the same contents, even if they have different paths, only one of them will be hydrated and the rest will be ignored.

In our case, this was noticed because we had a project with a file larger than the max blob size that would not get cached; then, when inevitably rebuilding it a second time, the hydrated dependency would miss files that a freshly-built dependency would have.

## The environment:

- A monorepo with two projects, A and B
  - B depends on A
  - At least two files in A have the same name and contents (but are in different directories)
  - B contains a file larger than 4MB, which is larger than the cache limit, causing it to not be saved
    - Alternatively, only build A the first time
- Remote caching
  - bazel-remote server (configuration in repo)
  - Zstd compression

## Steps to recreate:

1. Clone the repository.

   - Or create a monorepo with the previously-described environment.

2. Start the remote caching server.

   - Don't forget to change the domain name in the workspace file.

3. In the monorepo, run the commands:

   ```
   pnpm i
   moon ci :build
   ```

   This should succeed, although there should be a warning that B created a blob too large in size.

4. Again, in the monorepo, run the commands:
   ```
   git clean -f -x -d
   pnpm i
   moon ci :build
   ```
   A should be pulled from the cache, while B should have to rebuild again. This should cause B to fail its build process. If you check `dist` in A, some of the index.js files should be missing from their folders.
