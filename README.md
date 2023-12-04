# NDiff analyzer Gitlab pipeline

This is the repository for the Gitlab pipeline of the NDiff project.

## Setup

The contents of the `.gitlab-ci.yml` can be added to an existing CI-configuration to add 
the analyzer. <br>
To run the Gitlab jobs successfully the following CI Variables must be set:
* `$SEARCH_DIR` the root of the project that should be analysed
* `$NOTIFIER_API` the notification API to which the changes should be submitted

With these variables set, the jobs should run successfully on the default branch. The notification service will only be notified if there is a change of the API.

## Customization

It is possible to customise some aspects of the pipeline.

### Pipeline execution
Within the `.default_rules` there is a rule to such that the jobs only run
on the current default branch. This can be modified to include or exclude other branches by 
changing the `$CI_DEFAULT_BRANCH` variable.

e.g. `if: $CI_COMMIT_BRANCH == 'dev'`

### Output format of OpenAPI-diff
Also the output format of the openapi-diff analysis can be changed.
To make these changes to lines have to be altered:<br>
Change the output of the diff program:
```shell
CHANGED=$(java -jar /app/openapi-diff.jar ./diff_prev.json ./diff_head.json --state --html ./diff.html)
```
setting the output to a json file:
```shell
CHANGED=$(java -jar /app/openapi-diff.jar ./diff_prev.json ./diff_head.json --state --json ./diff.json)
```
The second step is to change the request type of the curl command:
```shell
curl --location --request POST "$NOTIFIER_API" --header "Content-Type: text/html" -d @./diff.html
```
here the content-type header must be changed:
```shell
curl --location --request POST "$NOTIFIER_API" --header "Content-Type: application/json" -d @./diff.json
```
With these changes the job can send any output to the notification service.

### When to submit changes
The OpenAPI-diff tool currently supports three different outputs: `no_changes`, `incompatible`, `compatible`.
In the current version of the pipeline, the output file of the tool is only send to the server if there are changes.
A user can further specify to only send a notification if there are incompatible changes:
```shell
if [ "$CHANGED" == "incompatible" ]; then
    echo "Reporting changes to users"
    curl --location --request POST "$NOTIFIER_API" --header "Content-Type: text/html" -d @./diff.html
else
    echo "No changes to report"
fi
```