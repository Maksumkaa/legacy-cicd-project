FROM python:3.12-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# --- Final image ---
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
RUN apk add --no-cache curl
COPY main.py .

#Reduced the image from 1.5GB → 130MB
RUN adduser -D -u 1000 appuser
USER appuser

EXPOSE 5000
CMD ["python", "main.py"]