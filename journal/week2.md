# Week 2 — Distributed Tracing

## Required homework

### Instrument with HoneyComb

#### Configure OpenTelemetry for Python-flask

I added OpenTelemetry to a backend service, and ensure that instrumentation data is being sent to Honeycomb

  - Add HoneyComb opentelemetry libraries in `requirements.txt` and used pip to install them. see code added below;

 ```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

### Initialize

  - Add the following updates to the app.py file for initializing a tracer and Flask instrumentation to send data to Honeycomb during Python Flask app initialization

```
# Initialize tracing with HoneyComb
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

  - For Docker Compose, configure the Honeycomb API environment variables

      ```
      OTEL_SERVICE_NAME: 'backend-flask'
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
      ```
   - To obtain a Tracer for creating spans, include the following lines of code in the services/home_activities.py file

      ```
      from opentelemetry import trace
      tracer = trace.get_tracer("tracer.name.here")
      ```
      
   - custom attribute created inside that span
      
      ```
      with tracer.start_as_current_span("home-activites-mock-data"):
      span = trace.get_current_span() # this will get the span
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat()) # this app.now attribute will show inside this span "home-activites-mock-data" , its data is the time now in ISO           foramt.
      ```
   
   - see honeycomb tracing result below;
<img width="1440" alt="Screenshot 2023-03-04 at 6 42 07 PM" src="https://user-images.githubusercontent.com/88699664/227735047-95cf13e7-ed02-40ad-9a66-0171dfd9f467.png">

<img width="1388" alt="Screenshot 2023-03-05 at 4 22 58 AM" src="https://user-images.githubusercontent.com/88699664/227735055-266b4732-e5fe-4511-87d8-c6491e64d18d.png">

  ### AWS X-RAY
   Instrument AWS X-Ray into Backend (Python-Flask) App
    - Add the line below to `requirements.txt` file and install it using `pip install -r requirements.txt`.
    
    
    aws-xray-sdk
    
   
      
   - To instrument your Flask application, start by adding the middleware to your application using the XRayMiddleware function in code. Then, configure a segment name        on the xray_recorder.

      ```
      ____
      from aws_xray_sdk.core import xray_recorder
      from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

      app = Flask(__name__)

      ___
      xray_url = os.getenv("AWS_XRAY_URL")
      xray_recorder.configure(service='cruddur-backend-flask', dynamic_naming=xray_url)
      XRayMiddleware(app, xray_recorder)
      ```

   - Create an `xray.json` file in the `backend-flask` directory and add the following lines:
```
    {
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

   - To generate a sampling rule, execute this command in the project directory:

       ```
       aws xray create-sampling-rule --cli-input-json file://aws/json/xray-sampling-rule.json
       ```
   ### Use docker-compose to configure and provision the X-Ray daemon and send data to the X-Ray API.
   
       
         xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "aws-region"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
      
      
   - Add these two environment variable to the backend (python-flask) in docker-compose file

      ```
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
      ```
   - see X-Ray traces within the AWS Console below;
      <img width="1299" alt="Screenshot 2023-03-05 at 6 47 45 AM" src="https://user-images.githubusercontent.com/88699664/227736092-14a146b9-0f73-4cf6-9e26-966b11eb3436.png">
      
   ### AWS CloudWatch Logs
   
   - added the following code to requirements.txt

     ```
     watchtower
     ```
     
     `cd` into the backend and run `pip install -r requirements.txt` to install watch tower
     
     insert the following code in `app.py` 
      
      ```
      import watchtower
      import logging
      from time import strftime
      ```
      - add the following code to `app.py` to set up a log group called Crudder
      
      ```
      # Configuring Logger to Use CloudWatch
      LOGGER = logging.getLogger(__name__)
      LOGGER.setLevel(logging.DEBUG)
      console_handler = logging.StreamHandler()
      cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
      LOGGER.addHandler(console_handler)
      LOGGER.addHandler(cw_handler)
      LOGGER.info("some message")
      ```
      - add the following code for error logging


      ```
      @app.after_request
      def after_request(response):
          timestamp = strftime('[%Y-%b-%d %H:%M]')
          LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
          return response
      ```
      
      - lets log something in an API endpoint



      ```
      LOGGER.info('Hello Cloudwatch! from  /api/activities/home')
      ```
      
      - Set the env var in your backend-flask for docker-compose.yml

      ```
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      ```
      
      
      
