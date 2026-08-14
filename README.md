# todo

A fast, local, single-user command-line todo manager written in TypeScript.

Tasks are stored as plain JSON in the directory you run the command from, so every project
folder gets its own independent todo list. No account, no network, no config file, no daemon.

> **Status: specified and planned, not yet implemented.**
> This README documents the intended behaviour defined in [SPEC.md](SPEC.md). The
> implementation is tracked in [tasks/todo.md](tasks/todo.md). Commands below will not run
> until Phase 2 of the plan is complete.

## Why

Most todo tools are either a full web app or a global list that mixes every project together.
This one is scoped to a directory. `cd` into a project, run `todo list`, and you see only that
project's tasks. Delete the folder and the list goes with it.

## Requirements

- Node.js 20.11 or newer
- npm

## Install

```bash
git clone <your-repo-url> my-app
cd my-app
npm install
npm run build
```

To make the `todo` command available globally on your machine:

```bash
npm link
```

Without `npm link`, run it directly:

```bash
node dist/cli.js list
```

## Usage

```
todo add <title>              Add a new task
todo list [--all]             List pending tasks; --all includes completed ones
todo done <id>                Mark a task as completed
todo delete <id>              Permanently remove a task
todo --help                   Show usage
todo --version               Show the version
```

### `todo add <title>`

Creates a new task with the next available id. The title should be quoted if it contains
spaces. Prints the id of the task it created.

```bash
todo add "write the parser"
```

### `todo list [--all]`

Prints your tasks, one per line, with the id you use for `done` and `delete`. By default only
pending tasks are shown; pass `--all` to include completed ones.

```bash
todo list
todo list --all
```

If there is no todo list in the current directory, it prints `No tasks yet.` and exits
cleanly — it will not create a file just because you looked.

### `todo done <id>`

Marks a task complete and records the completion time. The task disappears from `todo list`
but remains visible under `todo list --all`. Marking an already-completed task is harmless.

```bash
todo done 2
```

### `todo delete <id>`

Removes a task permanently and immediately — there is no confirmation prompt and no undo.
Ids of the remaining tasks do not change, and the deleted id is never reused.

```bash
todo delete 2
```

## Example session

Starting in an empty project directory:

```console
$ todo list
No tasks yet.

$ todo add "write the parser"
Added task 1: write the parser

$ todo add "write the tests"
Added task 2: write the tests

$ todo add "ship it"
Added task 3: ship it

$ todo list
  1  [ ]  write the parser
  2  [ ]  write the tests
  3  [ ]  ship it

$ todo done 1
Completed task 1: write the parser

$ todo list
  2  [ ]  write the tests
  3  [ ]  ship it

$ todo list --all
  1  [x]  write the parser
  2  [ ]  write the tests
  3  [ ]  ship it

$ todo delete 3
Deleted task 3: ship it

$ todo add "write the docs"
Added task 4: write the docs
```

Note the last step: the new task is id **4**, not 3. Ids are never recycled, so an id you
wrote down always refers to the same task.

## Where your data lives

A single file, `todos.json`, in the directory where you ran the command. It is created the
first time you add a task and is safe to read, edit by hand, commit, or delete.

```jsonc
{
  "version": 1,
  "nextId": 3,
  "tasks": [
    {
      "id": 1,
      "title": "write the parser",
      "done": true,
      "createdAt": "2026-08-14T10:00:00.000Z",
      "completedAt": "2026-08-14T10:30:00.000Z"
    },
    {
      "id": 2,
      "title": "write the tests",
      "done": false,
      "createdAt": "2026-08-14T10:05:00.000Z",
      "completedAt": null
    }
  ]
}
```

Writes are atomic — the file is written to a temporary path and then renamed into place — so
interrupting the command cannot leave you with a half-written list.

## Exit codes

Useful if you are calling `todo` from a script.

| Code | Meaning | Example |
|------|---------|---------|
| `0` | Success | The command did what you asked |
| `1` | User error | Unknown id, missing argument, unknown command |
| `2` | Data error | `todos.json` is missing required fields or is not valid JSON |

```bash
todo done 999
# → No task with id 999.   (stderr, exit code 1)
```

## Development

```bash
npm run dev -- add "task"   # Run from source without building
npm run build               # Typecheck and emit to dist/
npm run typecheck           # Typecheck only, no output
npm test                    # Build, then run the full test suite
npm run test:watch          # Re-run tests on change
npm run coverage            # Tests with coverage thresholds enforced
npm run lint                # ESLint
npm run lint:fix            # ESLint with autofix
npm run format              # Prettier
```

Tests use Node's built-in `node:test` runner. Unit tests cover the command logic as pure
functions, integration tests exercise the store against a real temporary directory, and
end-to-end tests spawn the built CLI and assert on stdout, stderr, and exit codes.

## Project layout

```
src/
  cli.ts            Entry point: argument parsing, exit codes, stderr
  commands/         One file per command, all pure functions
  store.ts          The only module that touches the filesystem
  format.ts         Rendering tasks for the console
  types.ts          Task and TodoFile shapes
tests/              Unit, integration, and end-to-end tests
SPEC.md             What we are building and why
tasks/plan.md       Implementation plan and task breakdown
tasks/todo.md       Ordered task checklist
```

## Scope

Deliberately not included: priorities, due dates, tags, subtasks, editing a task's title,
sync, multi-user support, and interactive mode. See the non-goals section of
[SPEC.md](SPEC.md) for the reasoning.

## License

MIT
