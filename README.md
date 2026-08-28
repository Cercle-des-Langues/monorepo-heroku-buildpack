Add as a first buildpack in the chain. Set the `MONOREPO_PROJECT_PATH` environment variable to point to the project root. It will be promoted to the slug's root, everything else will be erased. The following buildpack (e.g. nodejs) will finish slug compilation.

Optionally set `MONOREPO_MV_PATHS` to pull directories from elsewhere in the monorepo into the project before the flatten, so they survive it.

**Disclaimer:** The code may change without notice, so always pin to a specific github version. Provided as is. Forked from [timanovsky/subdir-heroku-buildpack](https://github.com/timanovsky/subdir-heroku-buildpack).

# How to use:
1. `heroku buildpacks:clear` if necessary
2. `heroku buildpacks:set https://github.com/Cercle-des-Langues/monorepo-heroku-buildpack`
3. `heroku buildpacks:add heroku/nodejs` or whatever buildpack you need for your application
4. `heroku config:set MONOREPO_PROJECT_PATH=projects/nodejs/frontend` pointing to what you want to be the project root.
5. Optionally `heroku config:set MONOREPO_MV_PATHS=...` (see below) to carry extra directories into the build.
6. Deploy your project to Heroku.

# Configuration

## `MONOREPO_PROJECT_PATH` (required)
Path, relative to the repo root, of the directory that should become the slug root. The build fails if it is unset or is not a directory in the build dir.

## `MONOREPO_MV_PATHS` (optional)
Comma-separated list of `src:dst` entries naming directories to move into the project *before* it is flattened to the build root.

- `src` and `dst` are both relative to the repo root.
- `dst` must be inside `MONOREPO_PROJECT_PATH`, otherwise it would not survive the flatten.
- Both sides are required.

```
MONOREPO_MV_PATHS="student_app:services/webapp/tmp/student_app"
MONOREPO_MV_PATHS="student_app:services/webapp/tmp/student_app,shared/proto:services/webapp/lib/proto"
```

Each moved `dst` is appended to a `.build_only_dirs` file at the build root, relative to the project root (i.e. to the build root after the flatten), so a trailing cleanup buildpack can prune those directories before slug packaging.

The build fails if an entry is not in `src:dst` form, either side is empty, `src` is not a directory in the build dir, `src` is inside `MONOREPO_PROJECT_PATH`, `dst` is outside `MONOREPO_PROJECT_PATH`, or `dst` already exists in the build dir.

# How it works
1. Moves each `MONOREPO_MV_PATHS` source to its `dst` inside the project directory, recording the `dst` path (relative to the project root) in `.build_only_dirs`.
2. Moves the project directory aside to `.subdir_stage` at the build root. (The build fails if `.subdir_stage` already exists — rename it.)
3. Erases everything else at the top level of the build dir, leaving the build dir root itself in place.
4. Promotes the contents of `.subdir_stage` to the build root.

Then normal Heroku slug compilation proceeds.
