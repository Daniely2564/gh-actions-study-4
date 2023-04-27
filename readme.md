# Controlling Execution Flow
- Sometimes you want to execute specific job at certain condition.

## Notes
Both **Jobs** and **Steps** can be executed via `if` field.

## Jobs

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

For if syntax, it always depends on the environment, so you can omit the syntax `${{ x }}` even though you can still use this.

You can additionally execute your own conditions with the help of Expressions.

