# Pull Request Author Checklist

Complete this checklist *before* requesting a review. Copy it into your PR description.

---

## Description

- [ ] I have written a clear PR title (format: `<type>(<scope>): <short summary>`)
- [ ] I have described *what* the change does and *why* it is needed
- [ ] I have linked the relevant ticket / issue
- [ ] I have noted any known trade-offs or alternative approaches I considered

## Code quality

- [ ] I have self-reviewed my diff and removed debug code, commented-out code, and `TODO`s that don't belong here
- [ ] I have kept the PR focused — unrelated changes are in a separate PR
- [ ] Variable, function, and class names are clear and consistent with the codebase
- [ ] Complex logic is explained with inline comments

## Tests

- [ ] I have added unit tests for new or changed behaviour
- [ ] I have added or updated integration/e2e tests where needed
- [ ] All existing tests pass locally
- [ ] Test coverage has not decreased (or I have explained why)

## Security

- [ ] No secrets, passwords, or PII are committed
- [ ] User input is validated and sanitised where applicable
- [ ] Any new dependencies have been reviewed for vulnerabilities

## Documentation

- [ ] Public APIs, configuration options, or environment variables are documented
- [ ] The README or wiki has been updated if the change affects how the system is run
- [ ] An ADR has been created if this introduces a significant architectural change

## Deployment

- [ ] Database migrations are backward-compatible (or a rollback plan exists)
- [ ] Feature flags are used if the change needs a dark launch
- [ ] Monitoring / alerting has been updated if new failure modes are introduced
- [ ] The change has been tested in a staging or preview environment

---

**Type of change** (tick all that apply)

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Refactor / tech debt
- [ ] Documentation only
- [ ] Infrastructure / CI change
