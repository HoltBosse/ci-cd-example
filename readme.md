# Continuous Integration/Deployment from Ubuntu dev to Ubuntu dev

This repository takes advantage of GitHub actions. Following is a rundown of the procedure for setting up CI/CD for an environment with a dev and production server.

## Notes

- `Workflow`: An automated process that will run one or more `jobs` as specified on a particular `event`. Worflows live at `.github/workflows` in a repository. 
- `Event`: A specific activity that may happen in a repository. Can be set up to trigger a `workflow` run.
- `Jobs`: Steps in a `workflow` executed by the same `runner`.
- `Actions`: Custom GitHub Actions application that performs a complex task that will generally be repeated a number of times.
- `Runners`: A server that runs `workflows`.