# Python Hello World Flask Application

A simple Flask web application that serves "Hello, World!" with Docker containerization and AWS deployment capabilities.

## Features

- ✅ Flask web application with a simple "Hello, World!" endpoint
- ✅ Production-ready Gunicorn WSGI server
- ✅ Docker containerization with `python:3.12-slim` base image
- ✅ Docker Compose for local development
- ✅ GitHub Actions CI/CD pipeline for AWS ECR and Elastic Beanstalk deployment

## Quick Start

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/jslmariano/python-hello.git
   cd python-hello
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run the Flask development server**
   ```bash
   python app.py
   ```
   
   The app will be available at `http://localhost:5000`

4. **Run with Gunicorn (production-like)**
   ```bash
   gunicorn -b 127.0.0.1:8000 wsgi:application --workers 2 --timeout 120
   ```
   
   The app will be available at `http://localhost:8000`

### Docker Development

1. **Build and run with Docker**
   ```bash
   docker build -t python-hello-app .
   docker run -p 8000:8000 python-hello-app
   ```

2. **Or use Docker Compose**
   ```bash
   docker compose up
   ```
   
   The app will be available at `http://localhost:8000`

## Production Deployment

### Prerequisites

Before deploying to AWS, ensure you have:

1. **AWS Account** with appropriate permissions
2. **ECR Repository** created named `python-hello`
3. **Elastic Beanstalk Application** and environment set up
4. **GitHub Secrets** configured:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`

### GitHub Actions Deployment

The CI/CD pipeline automatically:

- **On every push to main**: Builds and pushes Docker image to AWS ECR
- **On manual workflow dispatch**: Additionally deploys to Elastic Beanstalk

To trigger deployment:

1. Go to GitHub Actions in your repository
2. Select "Build and Deploy to AWS ECR and Elastic Beanstalk" workflow
3. Click "Run workflow"
4. Check "Deploy to Elastic Beanstalk" option
5. Click "Run workflow"

### Manual AWS Deployment

If you prefer manual deployment:

1. **Configure AWS CLI**
   ```bash
   aws configure
   ```

2. **Build and push to ECR**
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
   docker build -t python-hello .
   docker tag python-hello:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/python-hello:latest
   docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/python-hello:latest
   ```

3. **Create Dockerrun.aws.json for Elastic Beanstalk**
   ```json
   {
     "AWSEBDockerrunVersion": "1",
     "Image": {
       "Name": "<account-id>.dkr.ecr.us-east-1.amazonaws.com/python-hello:latest",
       "Update": "true"
     },
     "Ports": [
       {
         "ContainerPort": 8000,
         "HostPort": 80
       }
     ]
   }
   ```

4. **Deploy to Elastic Beanstalk**
   ```bash
   zip deploy.zip Dockerrun.aws.json
   aws elasticbeanstalk create-application-version --application-name python-hello --version-label v1 --source-bundle S3Bucket=<your-s3-bucket>,S3Key=deploy.zip
   aws elasticbeanstalk update-environment --environment-name python-hello-env --version-label v1
   ```

## Project Structure

```
python-hello/
├── app.py                    # Flask application
├── wsgi.py                   # WSGI entry point for Gunicorn
├── requirements.txt          # Python dependencies
├── Dockerfile               # Docker container configuration
├── docker-compose.yml       # Docker Compose configuration
├── .gitignore              # Git ignore rules
├── .github/
│   └── workflows/
│       └── deploy.yml      # GitHub Actions CI/CD pipeline
└── README.md               # This file
```

## Configuration

### Environment Variables

- `FLASK_ENV`: Set to `production` for production deployments (default in Docker)

### Docker Configuration

- **Base Image**: `python:3.12-slim`
- **Port**: `8000`
- **Gunicorn Configuration**: 2 workers, 120s timeout
- **Command**: `gunicorn -b 0.0.0.0:8000 wsgi:application --workers 2 --timeout 120`

### AWS Configuration

Update the following values in `.github/workflows/deploy.yml` as needed:

- `AWS_REGION`: AWS region (default: `us-east-1`)
- `ECR_REPOSITORY`: ECR repository name (default: `python-hello`)
- `EB_APPLICATION_NAME`: Elastic Beanstalk app name (default: `python-hello`)
- `EB_ENVIRONMENT_NAME`: Elastic Beanstalk environment (default: `python-hello-env`)

## Dependencies

- **Flask 3.0.0**: Web framework
- **Gunicorn 23.0.0**: Production WSGI server (security patched version)

## Security

This application uses the latest patched versions of all dependencies to ensure security best practices.

## License

This project is open source and available under the [MIT License](LICENSE).
