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

    
    

