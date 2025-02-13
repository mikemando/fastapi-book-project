# FastAPI Book Project

This project is a FastAPI application for managing books. It includes a **CI/CD pipeline** for automated testing and deployment and is served using **Nginx** as a reverse proxy.


### Installation & Setup
Fork & Clone the Repository:
```bash
git clone https://github.com/YOUR-USERNAME/fastapi-book-project.git
cd fastapi-book-project
```

### Set Up the Virtual Environment
Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On macOS/Linux
venv\Scripts\activate    # On Windows
```

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Run the Project Locally
```bash
uvicorn main:app
```
The API will be available at `http://127.0.0.1:8000`.

## API Endpoints

### Books

- `GET /api/v1/books/` - Get all books
- `GET /api/v1/books/{book_id}` - Get a specific book
- `POST /api/v1/books/` - Create a new book
- `PUT /api/v1/books/{book_id}` - Update a book
- `DELETE /api/v1/books/{book_id}` - Delete a book

### CI Pipeline Setup
Trigger: On **pull requests** to `main`.
#### Job: `test`
- Runs `pytest` to execute the existing tests.
- Fails if there are issues.
- Succeeds if all tests pass.

### Deployment Pipeline Setup
Trigger: On **merging** a pull request to `main`.
#### Job: `deploy`
- Automatically updates the deployed API with the latest changes.


Nginx is used as a **reverse proxy** to handle API requests.


## CI/CD Setup
The `.github/workflows` directory contains the CI/CD pipeline.
- **CI** (`test.yml`): Runs tests on pull requests.
- **CD** (`deploy.yml`): Deploys changes when merged to `main`.

## Deployment Instructions
###  Set Up an Amazon EC2 Instance
- Create an **Ubuntu EC2 instance**.
- Connect via SSH:
  ```bash
  ssh -i your-key.pem ubuntu@your-ec2-ip
  ```

### Install Required Packages on EC2
```bash
sudo apt update && sudo apt install -y python3-pip nginx
```

### Clone the Project on EC2
```bash
git clone https://github.com/YOUR-USERNAME/fastapi-book-project.git
cd fastapi-book-project
```

### Set Up a Virtual Environment & Install Dependencies
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Configure Nginx
Edit the Nginx configuration:
```bash
sudo nano /etc/nginx/sites-available/fastapi
```
Paste the following:
```nginx
server {
    listen 80;
    server_name YOUR_EC2_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Enable the configuration:
```bash
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

### Run the FastAPI App as a Service
Create a systemd service file:
```bash
sudo nano /etc/systemd/system/fastapi.service
```
Paste the following:
```ini
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/fastapi-book-project
ExecStart=/home/ubuntu/fastapi-book-project/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
```

### Test Deployment
```bash
curl http://YOUR_EC2_IP/
```
