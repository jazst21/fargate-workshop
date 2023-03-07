Context

I want to deploy a containerized API to serve requests and store the information in a database.
What We are Going to Build

In this module, we are going to deploy an API into Amazon ECS and running on AWS Fargate. The API will be integrated with Amazon DynamoDB to store all required information. We're going to use Amazon ELB, application load balancer (ALB). The endpoint for ALB will provide public access so any clients can access the API.

We're going to build an API to convert a Markdown input into HTML and store the requests in a database.
Diagram Architecture
![copilot fargate architecture diagram](../00-image/copilot-fargate-1.png.png "opilot fargate architecture diagram")
Module Dependencies

This module doesn't have any dependencies.
Lab Overview
While it's tempting to skip this segment, we encourage you to read through the entire instructions so you'll have a good understanding of how the application works.

This section will give you a brief information on application codes and also the required application environment.
Application environment

In this module you will bootstrap your first application environment.
Application codes review

The main file for the svc-api-markdown is app.py and it's written in Python using Flask framework. There are 2 routes available in this API:

    GET method — /health: Route for healtcheck done by ALB
    POST method — /api/markdown/process: endpoint to convert a Markdown input into HTML

Application requests flow

The request flow starts when the call goes to /api/markdown/process endpoint with method POST. This application accepts JSON payload input method with text as the variable input name.

If the application receives identifies the text in the JSON payload, it will assume the input is in Markdown format and convert into HTML output. Once it's successfully converted the input, it will save the data into DynamoDB through save_data function. If the response from the save_data function returns True, it will return the result along with HTTP status code 200. Otherwise, it will return error with HTTP status code 500.

Following code defines the flow:


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

This application has save_data function to save the input into database. The function simply uses update_item API to save the Markdown input and HTML output into DynamoDB with additional timestamp information.

def save_data(markdown, html):
    try:
        table = dynamodb.Table(
            os.getenv("<CHANGE_THIS_TO_YOUR_DATABASE_NAME>")
            )
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

Healthcheck Function

In app.py, you'll also find healthcheck function. It might be a simple function, however, without this function ALB will not understand if your application is working or not. If it doesn't accept proper health response from your application, it will assume your application can't receive requests.

@app.route('/health', methods=['GET'])
def healthcheck():
    data = {"status": "ok"}
    return jsonify(data), 200

