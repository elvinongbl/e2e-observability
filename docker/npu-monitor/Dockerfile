FROM python:3.11-slim

LABEL description="NPU Metrics Exporter for Intel NPU"

RUN apt-get update \
    && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
    
RUN git clone https://github.com/open-edge-platform/edge-ai-libraries \
    && mkdir -p /app/npu-monitor-tool \
    && cp -r edge-ai-libraries/tools/npu-monitor-tool/* /app/npu-monitor-tool/ \
    && rm -rf edge-ai-libraries

RUN ls -al /app/npu-monitor-tool
    
RUN pip install --no-cache-dir -r /app/npu-monitor-tool/requirements-exporter.txt

WORKDIR /app/npu-monitor-tool

CMD ["gunicorn", "-w", "1","-b", "0.0.0.0:9400", "npu-metrics-exporter:app"]
