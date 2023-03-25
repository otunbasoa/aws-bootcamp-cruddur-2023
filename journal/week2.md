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

