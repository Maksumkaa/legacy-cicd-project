FROM python:3.12-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# --- Final image ---
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY main.py .
COPY templates /app/templates

#Reduced the image from 1.5GB → 130MB
RUN useradd -m -u 1000 appuser
USER appuser

EXPOSE 5000
CMD ["python", "main.py"]