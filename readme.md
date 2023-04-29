# Controlling Execution Flow
- Sometimes you want to execute specific job at certain condition.

## Notes
Both **Jobs** and **Steps** can be executed via `if` field.

## Jobs

### Jobs Example 1
We can set up a job to run on failure.
```yml
jobs:
  # ...
  on-failure:
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: This would run on failure
        run: |
          echo "A job failed here this such detail ${{ github }}"
```
However, the previous job would end up failing since all of the jobs run in parallel. You would always want to make sure to run them sequentially using `needs` keyword.

```yml
  on-failure:
    if: failure()
    needs: # some build that this on-failure job waits for in case of failure.
    runs-on: ubuntu-latest
    steps:
      - name: ...
      # ...
```

The `needs` do not always have to run after an immediate job failure (e.g. previous job). This can be used at the end and watch for any failures coming from any previous jobs.

Example

If the pipeline follows as

```
        - Lint  -  -  -  - 
      /                     \
Start                          On Failure
      \                     /
        - Build - Deploy  - 
```

```yml
  needs : [Deploy, Lint]
```

In this case, even if the job, `Build` fails, the `On Failure` will be triggered.


## Steps
- Ignore errors via `continue-on-error` field
- Step result documentation: https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context

### Steps Example 1
Run a test only if a previous step fails. You would always need an id for a step in order to run a step based on its failure.

```yml
  steps
    - name: Test code
      run: npm run test
      id: test-components # adding unique id to reference this step.
    - name: Run if fails
      if: steps.test-components.outcome == 'failure'
      run: echo "Hey, I failed here :("
    - name: Upload test report
      uses: actions/upload-artifact@v3
      with:
        name: test-report
        path: test.json
```

Once you run this, you will find that this did not run. Reason -> our condition does check the result, but the default behavior is not to run the steps if any previous step fails. In order to handle this, we need to insert a new method. We can update the if condition as follows

```yml
    - name: Runs if fails
      if: failure() && steps.test-components.outcome == 'failure'
```

For if syntax, it always depends on the environment, so you can omit the syntax `${{ x }}` even though you can still use this.

You can additionally execute your own conditions with the help of Expressions.

### Steps Example 2 - Caching dependencies in case of failure

In case, you want to still download the dependencies and cache them in case of failure, you can activate step using actions/cache

```yml
  steps:
    - name: Will file for sure (Caches dependencies)
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install if not cached
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
```

## Continue On Error

In case, you want to run the job on failure, you can use `continue-on-error`. This will complete the job even if a step fails. 

## Special Conditional Functions

Function that gets evaluated when one of the below conditions meet.

- failure()
  - Returns `true` when any previous **Step** or **Job** fails
- success()
  - Returns `true` when a step or job succeeds
- always()
  - Always runs whether success/failure. Or even if it's cancelled.
- cancelled()
  - Returns true if workflow has been cancelled.

