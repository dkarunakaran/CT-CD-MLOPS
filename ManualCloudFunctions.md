This is a step by step process to setup Cloud Functions (CF) manually to run the Vertex AI pipeine. 

## Steps

1. Compile the pipeline and store it to the bucket
2. Enable CF and go to the functions page and click the create function.
3. Fill the necessary infos, more importantly, select the trigger type. In this, we chose the HTTP trigger. For the http trigger, we get URL which will call the cloud functions whenever we invoke the URL.
4. Choose "Allow unauthenticated invocations" as authentication and click next
5. Select the runtime as Python ans source code as inline.
6. Paste the below code to main.py
```
import functions_framework
import base64
import json
from google.cloud import aiplatform

PROJECT_ID = <your project>                   
REGION = <your region>           
PIPELINE_NAME = <your pipeline name>
GCS_BUCKET_NAME = <your bucket>
PIPELINE_ROOT = <your pipeline root>
pipeline_spec_uri = <your compiled pipeline file location>

@functions_framework.http
def hello_http(request):
    """HTTP Cloud Function.
    Args:
        request (flask.Request): The request object.
        <https://flask.palletsprojects.com/en/1.1.x/api/#incoming-request-data>
    Returns:
        The response text, or any set of values that can be turned into a
        Response object using `make_response`
        <https://flask.palletsprojects.com/en/1.1.x/api/#flask.make_response>.
    """
    request_json = request.get_json(silent=True)
    request_args = request.args

    if request_json and 'name' in request_json:
        name = request_json['name']
    elif request_args and 'name' in request_args:
        name = request_args['name']
    else:
        name = 'World'

    trigger_pipeline_run()
    
    return 'Pipeline run started'

def trigger_pipeline_run():

    PIPELINE_PARAMS = {
        "bucket_name":GCS_BUCKET_NAME,
        "display_name":PIPELINE_NAME,
        "project":PROJECT_ID,
        "location":REGION
    
    }

    # Create a PipelineJob using the compiled pipeline from pipeline_spec_uri
    aiplatform.init(
        project=PROJECT_ID,
        location=REGION,
    )
    job = aiplatform.PipelineJob(
        display_name=PIPELINE_NAME,
        template_path=pipeline_spec_uri,
        pipeline_root=PIPELINE_ROOT,
        enable_caching=False,
        parameter_values=PIPELINE_PARAMS
    )

    # Submit the PipelineJob
    job.submit()

```
8. Add following library name to requirements.txt
```
google-cloud-aiplatform
pyyaml
```
