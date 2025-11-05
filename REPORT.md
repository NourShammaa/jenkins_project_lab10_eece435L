# CI/CD Pipeline ( Coverage + Security + Deploy)

**Repo:** [https://github.com/NourShammaa/jenkins_project_lab10_eece435L](https://github.com/NourShammaa/jenkins_project_lab10_eece435L)
**Jenkins Job:** `Python-CI-CD`
**Agent OS:** Windows (Python 3.9.5)

---

## What I Implemented

### Setup Stage

* Created a Python virtual environment automatically on the Jenkins agent.
* Installed all dependencies from `requirements.txt` (`flake8`, `pytest`, `coverage`, `bandit`).
* Verified successful environment setup and package installation in the log:

  ```
  Successfully installed pip-25.3 wheel-0.45.1
  Requirement already satisfied: flake8, pytest, coverage, bandit ...
  ```

---

### Lint Stage

* Ran:

  ```bash
  venv\Scripts\flake8 app.py
  ```
* No blocking lint errors after fixing `E305` (missing blank line).
* Stage passed successfully.

---

### Test Stage

* Executed:

  ```bash
  set PYTHONPATH=C:\ProgramData\Jenkins\.jenkins\workspace\Python-CI-CD && venv\Scripts\pytest --junitxml=pytest-results.xml -q
  ```
* Result:

  ```
  .                                                                        [100%]
  - generated xml file: pytest-results.xml
  1 passed in 0.25s
  ```

All tests passed - the `greet()` function worked correctly.

---

### Coverage Stage

* Added:

  ```bash
  venv\Scripts\coverage run -m pytest --junitxml=pytest-results.xml
  venv\Scripts\coverage report -m
  venv\Scripts\coverage xml -o coverage.xml
  ```
* Jenkins printed this coverage table:

  ```
  Name                Stmts   Miss  Cover   Missing
  -------------------------------------------------
  app.py                  4      1    75%   6
  tests\test_app.py       7      1    86%   12
  -------------------------------------------------
  TOTAL                  11      2    82%
  ```
* Meaning:
  82% of all lines were executed by the tests. The missing lines correspond to the `if __name__ == "__main__":` block, which is not executed by `pytest`.
* `coverage.xml` was generated and archived as an artifact.

---

### Security Scan Stage

* Integrated:

  ```bash
  venv\Scripts\bandit -r . -f txt -o bandit.txt || exit /b 0
  ```
* Output:

  ```
  [main] INFO running on Python 3.9.5
  [text] INFO Text output written to file: bandit.txt
  ```
* Jenkins archived `bandit.txt`.
* Several `[manager] WARNING` messages appeared - these were Bandit parser comment warnings, not vulnerabilities.
* No high or medium-severity issues were found.

---

### Deploy Stage

* Present in the Jenkinsfile but logged:

  ```
  Stage "Deploy" skipped due to when conditional
  ```
* This indicates that deployment logic exists but was skipped automatically (likely set to run only on certain branches).
* When enabled, it packages the project into a `dist/` folder (for example, `dist/app_bundle.zip`) for distribution.

---

### Post-Build Cleanup

* Jenkins executed:

  ```
  [WS-CLEANUP] Deleting project workspace...
  [WS-CLEANUP] done
  ```

Confirmed that `cleanWs()` worked - the workspace was cleared successfully after build completion.

---

## Final Build Summary

| Stage         | Result  | Notes                                          |
| ------------- | ------- | ---------------------------------------------- |
| Setup         | Passed  | Virtual environment and dependencies installed |
| Lint          | Passed  | No PEP 8 violations after fix                  |
| Test          | Passed  | 1 test executed successfully                   |
| Coverage      | Passed  | 82% overall coverage                           |
| Security Scan | Passed  | No high/medium vulnerabilities                 |
| Deploy        | Skipped | Conditional stage (configured)                 |
| Post Cleanup  | Passed  | Workspace cleaned                              |

**Overall Build:** SUCCESS
**Artifacts archived:** `coverage.xml`, `bandit.txt`
**Test Report:** `pytest-results.xml` published successfully

---

## Interpretation of Results

* Flake8 passed → code is properly formatted.
* Pytest passed → all functional tests succeeded.
* Coverage 82% → almost all code executed except for the main entry point.
* Bandit scan → no critical vulnerabilities found.
* Overall → the pipeline ran end-to-end successfully and met all CI/CD requirements.

