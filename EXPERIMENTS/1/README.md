Hello. Few times I experimented with https://github.com/codenamev/ai-software-architect and wanted to see how it will be able to solve example issue discussed on Slack.

For setup, this is what I did manually:

1. I have `gh` installed and configured,
2. I have forked https://github.com/nonflux/build-definitions, cloned and created branch.

Then a series of commits and a description how I got them:

## Commit 1: 3179b34b99b3682a256fc720acf3d680088084b4

I manually installed https://github.com/codenamev/ai-software-architect by copying relevant files from other project and tweaking some to suite this repository more.

## Commit 2: e13a7c301000cfb04a9acd914118886d582d1e8f

I manually instructed AI to generate summary of generic guidelines docs to `.gemini/GEMINI.md` file to make it easier to use it.

## Commit 3: [38f61aaa1b54e4775b63c70652e3009d600f16b5](https://github.com/jhutar/noflux_build-definitions/commit/38f61aaa1b54e4775b63c70652e3009d600f16b5)

Here I let AI to create ADR for implementing the issue described in https://github.com/nonflux/build-definitions/issues/1

Note: Using `--prompt-interactive` and not `--prompt` (for fully detached mode) just to see what is going on. I have not interacted in any way. Using `--screen-reader` just to make copypasting easier.

Note: I intentionally used a separate sessions for each of the calls to simulate specialized agents performing each of these steps.

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Use the gh tool to read https://github.com/nonflux/build-definitions/issues/1 and output a summary of the core problem described in the issue.
3) Using the principles from step 1 and the problem context from step 2, create an ADR for implementing a solution to the issue in this forked repository.
4) Commit the new ADR.

"""
```

## Commit 4: [a4ecf42cadb1c61a7a6792f83653e244f9de634c](https://github.com/jhutar/noflux_build-definitions/commit/a4ecf42cadb1c61a7a6792f83653e244f9de634c)

Here I ran the plan through review:

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Check new ADR with \`git diff --name-only HEAD HEAD~1\`.
3) Run full architectural review for that ADR.
4) Implement changes suggested by the review to the ADR.
5) Commit review and changes in ADR as a single commit.

"""
```

## Commit 5: [9d15dd181463c4f1113b6372370483facf3f145c](https://github.com/jhutar/noflux_build-definitions/commit/9d15dd181463c4f1113b6372370483facf3f145c)

Implementation:

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Load ADR for this effort by \`git diff --name-only HEAD HEAD~2 | git diff --name-only HEAD HEAD~2\`.
3) Start implementing the ADR, commit after every logical step.
4) Once done, ensure all tests and checks as defined in .gemini/GEMINI.md are passing and if not, commit fixes in a separate commit.
5) Run the test and checks once again, ensure working tree is clean, push our branch and using the \`gh\` tool create a PR to fix https://github.com/nonflux/build-definitions/issues/1.

"""
```

## Conclusion

Result is totally wrong if I compare it with actual fix in https://github.com/konflux-ci/build-definitions/pull/3058

Result: https://github.com/jhutar/noflux_build-definitions/pull/1
