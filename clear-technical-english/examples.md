# Original writing examples

These examples illustrate this adaptation. They are not copied from the standard or certified as compliant.

## Remove filler

Before:
It is necessary to perform a restart of the application prior to attempting to use it again.

After:
Restart the application before you use it again.

## Put the condition first

Before:
Save the diagnostic logs before rollback in the event that the staging health check fails.

After:
If the staging health check fails, save the diagnostic logs before rollback.

## Keep a recommendation optional

Before:
Following the deployment, the operator should perform a health check.

After:
After the deployment, the operator should run a health check.

## Clarify a group of nouns

Before:
The service configuration update approval record is missing.

After:
The approval record for the service configuration update is missing.

## Split ordered actions

Before:
Open the settings panel, select the staging environment and save the configuration.

After:

1. Open the settings panel.
2. Select the staging environment.
3. Save the configuration.

## Keep uncertainty

Before:
It is possible that high memory use contributed to the service becoming unresponsive, although the cause has not been confirmed.

After:
High memory use may have contributed to the service failure. The cause is not confirmed.

## Keep terminology consistent

Before:
The queue consumer receives a message. The handler then checks its identifier. The worker records the result. All three names refer to the same component here.

After:
The queue consumer receives a message. It checks the message identifier and records the result.
