# Spring AI MCP Proof of Concept

This repository contains a Docker-based setup for running a GitHub Model Context Protocol (MCP) server locally. It's designed to work with Spring AI applications and provides a standardized way to interact with GitHub APIs through the MCP protocol.

## Features

- **GitHub MCP Server**: A containerized MCP server that provides GitHub API access
- **RESTful API**: HTTP endpoints for easy integration with Spring AI applications
- **Docker Compose**: Simple local deployment with Docker
- **Health Monitoring**: Built-in health checks and monitoring
- **Development Ready**: Hot-reload and development-friendly configuration

## Prerequisites

- Docker and Docker Compose installed
- GitHub Personal Access Token with appropriate scopes
- Node.js 18+ (for local development)

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/shesadri/spring-ai-mcp-poc.git
cd spring-ai-mcp-poc
```

### 2. Set Up Environment Variables

```bash
cp .env.example .env
```

Edit the `.env` file and add your GitHub Personal Access Token:

```env
GITHUB_PERSONAL_ACCESS_TOKEN=your_github_token_here
MCP_SERVER_PORT=3000
NODE_ENV=development
```

### 3. Generate GitHub Token

1. Go to [GitHub Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens)
2. Click "Generate new token (classic)"
3. Select the following scopes:
   - `repo` (Full control of private repositories)
   - `read:user` (Read user profile data)
   - `read:org` (Read organization data)
4. Copy the generated token to your `.env` file

### 4. Start the Services

```bash
docker-compose up -d
```

This will start:
- **GitHub MCP Server** on port 3000
- **Web UI** (optional) on port 8080

### 5. Verify the Setup

Check if the server is running:

```bash
curl http://localhost:3000/health
```

You should see:
```json
{
  "status": "healthy",
  "timestamp": "2025-06-01T04:22:00.000Z"
}
```

## API Endpoints

The GitHub MCP server exposes the following REST API endpoints:

### Repository Operations

- **Get Repository**: `GET /api/repos/{owner}/{repo}`
- **List User Repositories**: `GET /api/users/{username}/repos?type={all|owner|member}`

### Issues Operations

- **List Issues**: `GET /api/repos/{owner}/{repo}/issues?state={open|closed|all}`
- **Create Issue**: `POST /api/repos/{owner}/{repo}/issues`

### Pull Request Operations

- **List Pull Requests**: `GET /api/repos/{owner}/{repo}/pulls?state={open|closed|all}`

### Example API Calls

```bash
# Get repository information
curl http://localhost:3000/api/repos/octocat/Hello-World

# List user repositories
curl http://localhost:3000/api/users/octocat/repos

# List open issues
curl http://localhost:3000/api/repos/octocat/Hello-World/issues?state=open

# Create a new issue
curl -X POST http://localhost:3000/api/repos/octocat/Hello-World/issues \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test issue",
    "body": "This is a test issue created via MCP server",
    "labels": ["bug", "help wanted"]
  }'

# List pull requests
curl http://localhost:3000/api/repos/octocat/Hello-World/pulls
```

## Integration with Spring AI

This MCP server can be integrated with Spring AI applications. Here's an example configuration:

### Spring Boot Configuration

```yaml
# application.yml
spring:
  ai:
    mcp:
      servers:
        github:
          url: http://localhost:3000
          enabled: true
          timeout: 30s
```

### Java Integration Example

```java
@Service
public class GitHubMCPService {
    
    @Value("${mcp.github.base-url:http://localhost:3000}")
    private String baseUrl;
    
    private final RestTemplate restTemplate;
    
    public GitHubMCPService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    public Repository getRepository(String owner, String repo) {
        String url = baseUrl + "/api/repos/" + owner + "/" + repo;
        return restTemplate.getForObject(url, Repository.class);
    }
    
    public List<Issue> getIssues(String owner, String repo, String state) {
        String url = baseUrl + "/api/repos/" + owner + "/" + repo + "/issues?state=" + state;
        return Arrays.asList(restTemplate.getForObject(url, Issue[].class));
    }
    
    public Issue createIssue(String owner, String repo, CreateIssueRequest request) {
        String url = baseUrl + "/api/repos/" + owner + "/" + repo + "/issues";
        return restTemplate.postForObject(url, request, Issue.class);
    }
}
```

## Development

### Local Development Without Docker

1. Create the server directory:
```bash
mkdir mcp-github-server
cd mcp-github-server
```

2. Initialize the Node.js project:
```bash
npm init -y
npm install @modelcontextprotocol/sdk-nodejs @octokit/rest express cors dotenv
```

3. Copy the server implementation from the docker-compose.yml
4. Run locally:
```bash
npm start
```

### Monitoring and Logs

View server logs:
```bash
docker-compose logs -f github-mcp-server
```

Check container status:
```bash
docker-compose ps
```

## Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GITHUB_PERSONAL_ACCESS_TOKEN` | GitHub PAT for API access | Required |
| `MCP_SERVER_PORT` | Port for the MCP server | 3000 |
| `NODE_ENV` | Node.js environment | development |

### Docker Compose Services

- **github-mcp-server**: Main MCP server container
- **mcp-web-ui**: Optional web interface for testing

## Troubleshooting

### Common Issues

1. **Authentication Error**: Make sure your GitHub token has the required scopes
2. **Port Conflicts**: Change the `MCP_SERVER_PORT` in `.env` if port 3000 is in use
3. **Permission Denied**: Ensure Docker has permission to bind to the specified ports

### Debug Mode

Enable debug logging:
```bash
NODE_ENV=development docker-compose up
```

## Security Considerations

- Never commit your `.env` file with real tokens
- Use minimal required scopes for your GitHub token
- Consider using GitHub Apps for production deployments
- Implement rate limiting for production use

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with `docker-compose up`
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Related Projects

- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Spring AI](https://spring.io/projects/spring-ai)
- [GitHub REST API](https://docs.github.com/en/rest)
