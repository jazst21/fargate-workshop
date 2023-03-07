Lab Steps
This is a hands-on segment. All instructions are collapsed by default for easier reading. The important section you need to follow is marked with ✅. Additional information is marked with ℹ️.
✅ Step 0: Preparing codes
If you're running on your account and haven't downloaded your source code, please go to this link.

You need to work in the folder titled work-folder. If work-folder doesn't exist, please manually create it. It will be an empty folder since it's the first module. The work-folder is your primary folder.

You can copy the source folder of this module in source/mod1-deploy-api into the work-folder.

1
2
3
# From root folder `workshop`
cd work-folder
cp -rf ../source/mod1-deploy-api/svc-api-markdown .

Now in the work-folder folder, you will find files with following structure

1
2
3
4
5
work-folder/ # You should be in this folder
└──svc-api-markdown/
    ├── app.py
    ├── Dockerfile
    └── requirements.txt

✅ Step 1: Initialize application

When you use AWS Copilot, the first step you need to do is to initialize your application using copilot init command. You can enter interactive mode by just running copilot init, but in this module, we will use flags for consistency.

From your terminal in the root project folder, run the following command:

1
copilot init --app module1 --dockerfile svc-api-markdown/Dockerfile --name svc-api-markdown --type  "Load Balanced Web Service"

When Copilot asks to deploy into a test environment, choose N. We will setup an application environment in the next step.

1
Would you like to deploy a test environment? [? for help] (y/N)

This command creates a new application and store the manifest for svc-api-markdown in following path copilot/svc-api-markdown/manifest.yml. In the following section, we'll cover how to customize the manifest.

✅ Step 2: Configure path and healthcheck
Need help?

If you're having difficulty completing this step, you can also check the working code as reference: copilot/svc-api-markdown/manifest.yml

When you create a service in your application, you certainly need to define the path where the requests need to go through. For an example, you probably want to route all API requests to api/ — handled by API service — and to open the web page to web/ — handled by the web service.

Copilot provides you with flexibility to configure the path to suit your needs. As the ALB for Load Balanced Web Services is a shared resource, you can adjust the path to a specific service. You also need to configure health check path for your application.

To configure the HTTP application and health check paths, do the following:

    Open the manifest file located at copilot/svc-api-markdown/manifest.yml

    Find the section starts with http.

1
2
3
4
5
6
7
# Distribute traffic to your service.
http:
  # Requests to this path will be forwarded to your service.
  # To match all requests you can use the "/" path.
  path: '/'
  # You can specify a custom health check path. The default is "/".
  # healthcheck: '/'

    Adjust into following configuration

1
2
3
4
# File: copilot/svc-api-markdown/manifest.yml
http:
  path: 'api/markdown'
  healthcheck: '/health'

This configuration ensures the requests go to /api/markdown for your svc-api-markdown and the ALB will check the health of your application by making calls to /health.
Important

ALB will try to directly reach the healthcheck endpoint so in your application DO NOT add prefix — in this case don't set your healthcheck endpoint as /api/health as it won't work.

ℹ️ Step 3: Define task resources

In Amazon ECS, a Task is an instantiation of a task definition within your cluster. Task definition itself is a blueprint that defines the parameters on how your application works. In the following section, you can define how much CPU, memory, and how many tasks you want to run for your application.

1
2
3
4
cpu: 256 # Number of CPU units for the task.
memory: 512 # Amount of memory in MiB used by the task.
count: 1 # Number of tasks that should be running in your service.
exec: true # Enable running commands in your container.

You only need to review this section and don't need to do anything else.

ℹ️ Step 4: Scaling behaviour

Another section that you will need in your service is configuring the scaling behaviour of your application.

Following configuration is an example on how you can scale your application from minimum 1 task to maximum 10 tasks if the CPU percentage value of your service resulting in average 70%.

1
2
3
count:
  range: 1-10
  cpu_percentage: 70

✅ Step 5: Initialize environment

At this stage, you have initialized your service as a new application. However, you don't have any environment for your service yet.

The next step you need to is to configure an environment called staging. Run the following command to create an environment for your svc-api-markdown:

1
copilot env init --name staging --default-config --profile default

☕️ Coffee Break
This will take you around 1-2 minutes.

Above command will create an environment called staging with the named profile default in your AWS credentials configuration. Copilot will create the manifest file for your staging environment in the following path: copilot/environments/staging/manifest.yml. Similar to other manifests, you can also customize the manifest based on your needs.

With the --profile default flag, you're configuring a new VPC and multi-AZ, with each has a public and private subnets. You also can use existing VPC by using --import-vpc-id flag, however, creating with the --default-config is a best practice and generally recommended.

The next step is to deploy environment using following command:

1
copilot env deploy --name staging

☕️ Coffee Break
This will take you around 1-2 minutes.

✅ Step 6: Integrate with database

This application requires an integration with a database to work properly. In this step, you will configure DynamoDB and modify the application code.

To create DynamoDB resource with Copilot, run the following command:

1
copilot storage init -t DynamoDB -n markdown-table --partition-key ID:S --no-lsi --no-sort -w svc-api-markdown

This command will create a manifest file for a DynamoDB table named markdown-table with ID as partition key with type string, without LocalSecondaryIndex and Sort Key attached to svc-api-markdown. The manifest file can be found at copilot/svc-api-markdown/addons/markdown-table.yml.

✅ Step 7: Modify application codes
Need help?

If you're having difficulty completing this step, you can also check the working code as reference: svc-api-markdown/app.py

Once you run the command, you will receive an output as below:

1
Update svc-api-markdown code to leverage the injected environment variable MARKDOWNTABLE_NAME.

The next step is to modify our application code to use the environment variable.

Open svc-api-markdown/app.py and change the following line:

1
table = dynamodb.Table(os.getenv("<CHANGE_THIS_TO_YOUR_DATABASE_NAME>"))

Into:

1
table = dynamodb.Table(os.getenv("MARKDOWNTABLE_NAME"))

✅ Step 8: Deployment!

The next step is to deploy the application. Run the following command to deploy your first application:

1
copilot svc deploy --name svc-api-markdown --env staging

☕️ Coffee Break
This will take you around 3-8 minutes.

🥳 Congrats!

You just deployed your first application with AWS Copilot!
✅ Step 9: Test applications

To test your application, you can get the URL of your API by running:

1
copilot svc show --name svc-api-markdown

Get the URL in Route section from the output:

1
2
3
4
5
Routes

  Environment  URL
  -----------  ---
  staging      http://APP.ap-southeast-1.elb.amazonaws.com

Now, before your test your application, you need to remember that we've defined /api/markdown for your service route, and to process Markdown conversion, you need to use process path. All we need is just to form the right URL: http://<URL>/api/markdown/process.

You can test the URL by using curl:

1
curl -X POST <URL>/api/markdown/process -d '{"text":"# Hello world from AWS Copilot!"}' --header "Content-type: application/json"

If everything works, you will see the following output:

1
2
3
4
{
  "html": "<h1>Hello World from AWS Copilot</h1>",
  "markdown": "# Hello World from AWS Copilot"
}

You can also check from DynamoDB console if the record is created. You need to go to Amazon DynamoDB console, and find the table. If you follow the steps, you'll have a table named as module1-staging-svc-api-markdown-markdown-table.

✅ Step 10: Monitor application logs

Once you have your application deployed, you can also run some additional commands that useful if you're doing monitoring.

To see the overall status of your application

1
copilot app show

To see the logs:

1
copilot svc logs --name svc-api-markdown --env staging

To watch the logs:

1
copilot svc logs --name svc-api-markdown --env staging --follow

What We Have Learned

    Use copilot init to start initializing your apps
    All IaC manifests are stored in ./copilot/ folder
    Use copilot storage to connect Amazon S3 bucket, Amazon DynamoDB, and Amazon Aurora
    One backing resource is only meant for one service. Connect through APIs for microservices
    Use copilot deploy to deploy apps
    Use copilot status to check overall status
    Use copilot svc show to get detailed information
    Use copilot svc logs (with --follow) to get all the logs from your applications

Cleaning Up
Before you delete the application, please check other modules that might have dependency with this module. If other modules have a dependency with this module, you need to redo from begining of this module.

Once you're done working in this module, you can remove all the resources by running following command in the root folder of the module:

1
2
copilot svc delete --name svc-api-markdown
copilot env delete --name staging

If you'd prefer to remove the entire resources including the application, run the following command.

1
copilot app delete

Code References

Below is the list of working codes that you can use as reference.
svc-api-markdown/app.py
Expand to see working code

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
# MIT No Attribution

# Copyright 2022 AWS

# Permission is hereby granted, free of charge, to any person obtaining a copy of this
# software and associated documentation files (the "Software"), to deal in the Software
# without restriction, including without limitation the rights to use, copy, modify,
# merge, publish, distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
# INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
# PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
# HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
# OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
# SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

from flask import Flask, request, jsonify
import markdown
import os
import boto3
import uuid
from datetime import datetime
import logging


ENV_NAME = os.getenv("COPILOT_ENVIRONMENT_NAME")
LOGGING_LEVEL = logging.INFO if ENV_NAME == "production" else logging.DEBUG
DEBUG_MODE = False if ENV_NAME == "production" else True
logging.basicConfig(level=LOGGING_LEVEL)

app = Flask(__name__)

dynamodb = boto3.resource('dynamodb')

def save_data(markdown, html):
    try:
        table = dynamodb.Table(os.getenv("MARKDOWNTABLE_NAME"))
        id = str(uuid.uuid4())
        table.update_item(
            Key={'ID': id},
            UpdateExpression="set message_markdown=:markdown, message_html=:html, request_date=:sts",
            ExpressionAttributeValues={
                ':markdown': markdown,
                ':html': html,
                ':sts': datetime.now().strftime("%m-%d-%Y %H:%M:%S")
            })
        return True
    except Exception:
        logging.exception("Error on saving data into DynamoDB", exc_info=True)
        return False


@app.route('/health', methods=['GET'])
def healthcheck():
    data = {"status": "ok"}
    return jsonify(data), 200


@app.route('/api/markdown/process', methods=['POST'])
def to_markdown():
    response_data = {}
    response_status = 500    
    try:
        logging.info("Received request: {}".format(request.json))
        if "text" not in request.json:
            response_data = {"error":"No accepted parameters"}
            response_status = 400
            return jsonify(response_data), response_status

        if "text" in request.json:
            input_markdown = request.json['text']
            html = markdown.markdown(input_markdown)
            db_status = save_data(input_markdown, html)
            if db_status:
                response_data = {"markdown": input_markdown, "html": html}
                response_status = 200
            else:                
                response_data = {"error": "Unable to save data to database"}
                response_status = 500
            return jsonify(response_data), response_status
    except Exception as e:
        logging.error("Error on handling request {}".format(e), exc_info=True)
        return jsonify(response_data), response_status

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=9090, debug=DEBUG_MODE)

copilot/svc-api-markdown/manifest.yml
Expand to see working code

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
# The manifest for the "svc-api-markdown" service.
# Read the full specification for the "Load Balanced Web Service" type at:
#  https://aws.github.io/copilot-cli/docs/manifest/lb-web-service/

# Your service name will be used in naming your resources like log groups, ECS services, etc.
name: svc-api-markdown
type: Load Balanced Web Service

# File: copilot/svc-api-markdown/manifest.yml
http:
  path: 'api/markdown'
  healthcheck: '/health'


# Configuration for your containers and service.
image:
  # Docker build arguments. For additional overrides: https://aws.github.io/copilot-cli/docs/manifest/lb-web-service/#image-build
  build: svc-api-markdown/Dockerfile
  # Port exposed through your container to route traffic to it.
  port: 9090

cpu: 256       # Number of CPU units for the task.
memory: 512    # Amount of memory in MiB used by the task.
count: 1       # Number of tasks that should be running in your service.
exec: true     # Enable running commands in your container.

# storage:
  # readonly_fs: true       # Limit to read-only access to mounted root filesystems.
 
# Optional fields for more advanced use-cases.
#
#variables:                    # Pass environment variables as key value pairs.
#  LOG_LEVEL: info

#secrets:                      # Pass secrets from AWS Systems Manager (SSM) Parameter Store.
#  GITHUB_TOKEN: GITHUB_TOKEN  # The key is the name of the environment variable, the value is the name of the SSM parameter.

# You can override any of the values defined above by environment.
#environments:
#  test:
#    count: 2               # Number of tasks to run for the "test" environment.
#    deployment:            # The deployment strategy for the "test" environment.
#       rolling: 'recreate' # Stops existing tasks before new ones are started for faster deployments.

Click next to have fun with the next module 👉🏻