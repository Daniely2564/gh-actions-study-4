# Controlling Execution Flow
- Sometimes you want to execute specific job at certain condition.

## Notes
Both **Jobs** and **Steps** can be executed via `if` field.

## Jobs

## Steps
- Ignore errors via `continue-on-error` field

### Steps Example
Run a test only if a previous step fails

```yml
  steps
    - name: Test code
      run: npm run test
    - name: Run if fails
      
```

You can additionally execute your own conditions with the help of Expressions.

