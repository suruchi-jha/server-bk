# Blog Application

A Spring Boot blog application with user authentication, blog management, and genre categorization.

## Prerequisites

- Java 21
- Maven 3.9+
- Docker (optional)
- MySQL database (or use H2 for development)

## Environment Setup

1. Create a `.env` file in the root directory with your database credentials:
```env
SPRING_DATASOURCE_URL=jdbc:mysql://your-mysql-host:port/database_name?ssl-mode=REQUIRED
SPRING_DATASOURCE_USERNAME=your_username
SPRING_DATASOURCE_PASSWORD=your_password
AI_PROVIDER=gemini
AI_MODEL=gemini-2.5-flash
AI_GEMINI_API_KEY=your_gemini_api_key
```

Keep `.env` private. It is excluded from the Docker build context and is supplied to the running container with `--env-file`.

## Running the Application

### Option 1: Local Development (Recommended)

1. **Start the application:**
```bash
./mvnw spring-boot:run
```

2. **Access the application:**
- Main application: http://localhost:8080
- H2 Console: http://localhost:8080/h2-console
- API endpoints: http://localhost:8080/api/

### Option 2: Using Docker

1. **Build the Docker image:**
```bash
docker build -t blog-application .
```

2. **Run the container:**
```bash
docker run -p 8080:8080 --env-file .env blog-application
```

3. **Or run with environment variables:**
```bash
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_URL="your_database_url" \
  -e SPRING_DATASOURCE_USERNAME="your_username" \
  -e SPRING_DATASOURCE_PASSWORD="your_password" \
  -e AI_PROVIDER="gemini" \
  -e AI_MODEL="gemini-2.5-flash" \
  -e AI_GEMINI_API_KEY="your_gemini_api_key" \
  blog-application
```

### Verify the container

After the container starts, verify the server before opening the frontend:

```bash
curl http://localhost:8080/api/auth/check
```

This should return JSON with `"authenticated":false`. AI endpoints require a logged-in session, so test them from the frontend after signing in.

## Docker Hub Deployment

### Building and Pushing to Docker Hub

1. **Login to Docker Hub:**
```bash
docker login
```

2. **Build the image:**
```bash
docker build -t demo-application .
```

3. **Tag the image (if not done during build):**
```bash
docker tag demo-application suru1808/demo-application:latest
```

4. **Push to Docker Hub:**
```bash
docker push suru1808/demo-application:latest
```

### Pulling and Running from Docker Hub

1. **Pull the image:**
```bash
docker pull suru1808/demo-application:latest
```

2. **Run the container:**
```bash
docker run -p 8080:8080 --env-file .env suru1808/demo-application:latest
```

## API Endpoints

### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `GET /api/auth/check` - Check authentication status
- `POST /api/auth/logout` - User logout

### Blogs
- `GET /api/blogs` - Get all blogs (authenticated)
- `GET /api/blogs/popular` - Get popular blogs
- `GET /api/blogs/genre/{genreId}` - Get blogs by genre
- `GET /api/blogs/{id}` - Get specific blog

### Genres
- `GET /api/genres` - Get all genres
- `GET /api/genres/{id}` - Get specific genre

### Users
- `GET /api/users/search?q={query}` - Search users
- `GET /api/users/{username}` - Get user profile

## Testing with Postman/curl

### Register a new user:
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123"
  }'
```

### Login:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```

### Check authentication:
```bash
curl -X GET http://localhost:8080/api/auth/check
```

## Common Issues & Troubleshooting

### 1. 415 Unsupported Media Type Error
**Problem:** API returns 415 when sending requests
**Solution:** Ensure you're sending `Content-Type: application/json` header

**Postman Setup:**
- Method: POST
- URL: http://localhost:8080/api/auth/register
- Headers: `Content-Type: application/json`
- Body: Raw JSON format

### 2. 405 Method Not Allowed
**Problem:** GET request to POST-only endpoint
**Solution:** Use POST method for registration/login endpoints

### 3. Database Connection Issues
**Problem:** Application can't connect to database
**Solution:** 
- Check `.env` file exists and has correct credentials
- Ensure database is running and accessible
- Verify connection string format

### 4. Docker Build Issues
**Problem:** Build fails or image is too large
**Solution:**
- Add `.dockerignore` file to exclude unnecessary files
- Use multi-stage builds (already implemented)

### 5. Port Already in Use
**Problem:** Port 8080 is already occupied
**Solution:**
```bash
# Find process using port 8080
netstat -ano | findstr :8080

# Kill the process (replace PID with actual process ID)
taskkill /PID <PID> /F

# Or use a different port
docker run -p 8081:8080 your-username/blog-application
```

## Development Commands

### Maven Commands
```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Package application
mvn clean package

# Run application
mvn spring-boot:run

# Skip tests during build
mvn clean package -DskipTests
```

### Docker Commands
```bash
# List all images
docker images

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi <image_id>

# View container logs
docker logs <container_id>

# Execute command in running container
docker exec -it <container_id> /bin/bash
```

## Project Structure

```
src/
├── main/
│   ├── java/com/blog/
│   │   ├── config/          # Configuration classes
│   │   ├── controller/      # MVC and REST controllers
│   │   ├── model/          # Entity models
│   │   ├── repository/     # Data repositories
│   │   └── service/        # Business logic services
│   └── resources/
│       ├── application.properties
│       └── static/         # Static web resources
└── test/                   # Test classes
```

## Configuration

### Database Configuration
The application uses MySQL by default but can be configured for H2 in development.

### Security Configuration
- CSRF disabled for API endpoints
- Session-based authentication
- CORS enabled for frontend integration

### Logging
Debug logging is enabled for:
- Spring Security
- Spring Web
- Application packages
- Spring Session

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the MIT License.
