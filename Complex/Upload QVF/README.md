# Complex workflows

This example recipe shows how to handle publishing new version of an app.

Lets image we are moving an app from pre-prod to prod environment. First we have to upload the app to prod (with or without data). Once the app is uploaded we can directly overwrite the production app and then start the reload to bring the prod data. If this is overnight operation might not be a problem but if this is done during users active hours then until the app is reloaded the users will either view pre-prod data or an empty app (while the reload is complete).

Because of the above our workflow is to:

- upload the app
- reload it
- when the reload completes (successfully) publish the app

The whole operation is contained in two runbooks:

- `upload-and-set-tasks`

Contains 6 tasks and will upload the app, set reload tasks and will start the reload task

- `Upload app` - upload qvf
- `Create reload task` - create task that will reload the uploaded app
- `Get published app` - get details about the app that the uploaded app will overwrite (aka the current published app)
- `Run Automatiqal runbook to publish an app` - create external task that will start `publish-app` runbook to publish the app. The task will set the inline variables that are used in `publish-app` runbook
- `Trigger external task on reload success` - add `success` trigger to the reload task (created in `Create reload task`). This trigger will be triggered when `Create reload task` task finish successfully .
- `Start reload of "My uploaded app"` - the final step is to start the reload task

- `publish-app`

Will publish the app into dedicated stream when the reload task is complete

The whole process will leave one "parasite" external task that can be removed after the app is published. It can be removed:

- manually
- automatically - once the app is published we can tag the external task with specific tag (`DELETE ME` for example) and have another runbook that runs on scheduled basis and clear all objects that are tagged.
