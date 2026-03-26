Now bit more direct approach (reused first two commits from previous attempt).

## Commit 3: [fe27c8cd5fa351c8af227b4a04acaae2007daa8c](https://github.com/jhutar/noflux_build-definitions/commit/fe27c8cd5fa351c8af227b4a04acaae2007daa8c)

Investigate and plan change.

Note: This can possibly run with `--approval-mode plan` + gh execution, possibly improving security.

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Use the \`gh\` tool to read https://github.com/nonflux/build-definitions/issues/1 and review all the attachements.
3) Investigate related codebase and create a implementation plan in .architecture/plans/.
4) Commit the plan.

"""
```

## Commit 4: many, see commits in this branch

Implementation.

```
$ gemini --screen-reader --yolo --prompt-interactive """

Hello Gemini. Please complete the following steps in order:

1) Read .gemini/GEMINI.md into your context.
2) Check implementation plan with \`git diff --name-only HEAD HEAD~1\`.
3) Implement changes, commit after each logical change.
4) Once changes were implmented, do a full architectural review of all the changes and fix importat issues the review found and commit these as sepparate commit(s).
5) Run all tests and checks as described in .gemini/GEMINI.md and if needed, commit changes in new commit.
6) Ensure workdir is clean and create a PR to fix https://github.com/nonflux/build-definitions/issues/1 .

"""
```

* [7ba086422668d39d6d8e905afa2459596908c979](https://github.com/jhutar/noflux_build-definitions/commit/7ba086422668d39d6d8e905afa2459596908c979)
* [e91b83f90d1716450261e6075fe5a5dd48bc1800](https://github.com/jhutar/noflux_build-definitions/commit/e91b83f90d1716450261e6075fe5a5dd48bc1800)
* [267b9dd15252edc633cce0da538153396eed6fba](https://github.com/jhutar/noflux_build-definitions/commit/267b9dd15252edc633cce0da538153396eed6fba)
* [257b5a85d5f7a06fa3b74e1e97470a6983fdf3d0](https://github.com/jhutar/noflux_build-definitions/commit/257b5a85d5f7a06fa3b74e1e97470a6983fdf3d0)

## Conclusion

Looks OK, but additional unneeded changes were commited. References not updated.

Result: https://github.com/jhutar/noflux_build-definitions/pull/2
