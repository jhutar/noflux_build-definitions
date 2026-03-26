Trying in a straight-forward way:

## Step 1: Create docs (one time job)

Commit [6a1d6fe4b93b0d2b721712cb401ace7a13cff51a](https://github.com/jhutar/noflux_build-definitions/commit/6a1d6fe4b93b0d2b721712cb401ace7a13cff51a)

Just to reformulate existing docs to `.gemini/GEMINI.md`.

## Step 2: Do the work

Note: Using `--prompt-interactive` and not `--prompt` (for fully detached mode) just to see what is going on. I have not interacted in any way. Using `--screen-reader` just to make copypasting easier.

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Use the \`gh\` tool to read https://github.com/nonflux/build-definitions/issues/1 and review all the attachments.
3) Investigate related codebase thoroughly, implement changes, commit after each logical change.
5) Run all tests and checks as described in .gemini/GEMINI.md and if needed, commit changes in new commit.
6) Ensure workdir is clean and create a PR to fix https://github.com/nonflux/build-definitions/issues/1 .

"""
```

Commit [128ad6e2a05a9f22ec3bc75338afc8037ceeb1a6](https://github.com/jhutar/noflux_build-definitions/commit/128ad6e2a05a9f22ec3bc75338afc8037ceeb1a6).

## Conclusion

Did the OK (?) change in a stepaction, but did not updated references.

Result: https://github.com/nonflux/build-definitions/pull/2
