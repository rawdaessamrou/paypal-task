# Base image
FROM alpine:3.18
 
# Install Python and pip
RUN apk add --no-cache python3 py3-pip
 
# Set working directory
WORKDIR /app
 
# Copy dependencies file and install
COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt
 
# Copy application source code
COPY . .
 
# Expose the Flask default port
EXPOSE 5000
 
# Run the Flask app with Gunicorn for production
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
 