# Contributing

Interested in contributing to the Hartshorn? Want to report a bug?
Have a question? Before you do, please read the following guidelines.

## Submission context

### Found a bug?

If you found a bug in the source code, you can help us by submitting an issue
to the [issue tracker] in our GitHub repository. Even better, you can submit
a Pull Request with a fix. However, before doing so, please read the
[submission guidelines].

  [issue tracker]: https://github.com/GuusLieben/Hartshorn/issues
  [submission guidelines]: #submission-guidelines

### Missing a feature?

You can request a new feature by submitting an issue to our GitHub Repository.
If you would like to implement a new feature, please submit an issue with a
proposal for your work first, to be sure that it is of use for everyone, as
Hartshorn is highly opinionated. Please consider what kind of change it is:

* For a **major feature**, first open an issue and outline your proposal so
  that it can be discussed. This will also allow us to better coordinate our
  efforts, prevent duplication of work, and help you to craft the change so
  that it is successfully accepted into the project.

* **Small features and bugs** can be crafted and directly submitted as a Pull
  Request. However, there is no guarantee that your feature will make it into
  the `hartshorn-main`, as it's always a matter of opinion whether if benefits 
  the overall functionality of the project.

## Submission guidelines

### Submitting an issue

Before you submit an issue, please search the issue tracker, maybe an issue for
your problem already exists and the discussion might inform you of workarounds
readily available.

We want to fix all the issues as soon as possible, but before fixing a bug we
need to reproduce and confirm it. In order to reproduce bugs we will
systematically ask you to provide a minimal reproduction scenario using the
custom issue template. Please stick to the issue template.

Unfortunately we are not able to investigate / fix bugs without a minimal
reproduction scenario, so if we don't hear back from you we may close the issue.

### Submitting a Pull Request (PR)
Pull Requests always follow the [Pull Request Template](https://github.com/GuusLieben/Hartshorn/blob/hartshorn-main/PULL_REQUEST_TEMPLATE.md).

#### General
- Try to limit pull requests to a few commits which resolve a specific issue
- Make sure commit messages are descriptive of the changes made
- All proposed changes must be reviewed and approved by at least one organization member
- Ensure all new code is documented according to the [JavaDoc styleguide](https://github.com/GuusLieben/Hartshorn/blob/hartshorn-main/CONTRIBUTING.md#javadocs)
- Describe the proposed changes with a relevant motivation and additional context
- Link to the original issue(s) which your changes relate to

#### Additional information
- Indicate what type of changes your proposal makes
- Indicate if your proposed changes contain breaking changes
- Indicate if your proposed changes requires additional documentation (apart from JavaDocs)

#### Testing
- Every pull request will be tested by GitHub workflows
- Test cases should be provided when appropriate. This includes making sure new features have adequate code coverage
- If your proposed changes were tested using practical use-cases (run testing) describe your test configuration
- The pull request must pass all GitHub workflow checks before being accepted

