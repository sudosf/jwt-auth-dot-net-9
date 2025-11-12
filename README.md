# JWT Authentication API - .NET 9

A modern ASP.NET Core 9 REST API demonstrating JWT (JSON Web Token) authentication with refresh token support, role-based authorization, and SQL Server integration.

## Features

- ✅ **User Registration & Login** - Create accounts and authenticate with username/password
- ✅ **JWT Token Authentication** - Secure token-based authentication
- ✅ **Refresh Token Support** - Extend sessions with automatic token refresh
- ✅ **Role-Based Authorization** - Restrict endpoints to specific user roles (e.g., Admin)
- ✅ **Password Hashing** - Secure password storage using ASP.NET Identity
- ✅ **Entity Framework Core** - Database ORM for SQL Server
- ✅ **API Documentation** - Scalar UI for interactive API exploration
- ✅ **Database Migrations** - Version-controlled database schema

## Technology Stack

- **.NET 9** - Latest .NET framework
- **ASP.NET Core** - Web framework
- **Entity Framework Core 9** - ORM for database operations
- **SQL Server** - Relational database
- **JWT Tokens** - Stateless authentication
- **Scalar** - OpenAPI/Swagger documentation UI

## Project Structure

```
JwtAuthDotNet9/
├── Controllers/
│   └── AuthController.cs          # API endpoints for auth operations
├── Services/
│   ├── AuthService.cs             # Business logic for authentication
│   └── IAuthService.cs            # Service interface
├── Data/
│   └── UserDbContext.cs           # Entity Framework database context
├── Entities/
│   └── User.cs                    # User database entity
├── Models/
│   ├── UserDto.cs                 # Login/register request model
│   ├── TokenResponseDto.cs        # Token response model
│   └── RefreshTokenRequestDto.cs  # Refresh token request model
├── Migrations/                    # Database schema history
├── Properties/
│   └── launchSettings.json        # Run configurations
├── appsettings.json               # Configuration
├── Program.cs                     # Application startup
└── JwtAuthDotNet9.csproj         # Project file

```

## Prerequisites

- **.NET 9 SDK** - [Download](https://dotnet.microsoft.com/download/dotnet/9.0)
- **SQL Server** - Local or Docker instance
- **Visual Studio Code** or **Visual Studio** (optional)

## Installation & Setup

### 1. Clone/Open the Project

### 2. Configure Database Connection

Edit `appsettings.json` and update the connection string:

```json
{
  "ConnectionStrings": {
    "UserDatabase": "Server=localhost,1433;Database=UserDb;User Id=sa;Password=YOUR_PASSWORD;TrustServerCertificate=true;"
  },
  "AppSettings": {
    "Token": "YourSuperSecureKeyAtLeast32CharactersLong!!!",
    "Issuer": "MyAwesomeApp",
    "Audience": "MyAwesomeAudience"
  }
}
```

**⚠️ Important Security Notes:**

- Change the `Token` to a long, random secret key (used for signing JWTs)
- Use a strong database password
- Store sensitive configuration in Azure Key Vault or similar in production
- Never commit real credentials to version control

### 3. Apply Database Migrations

```bash
dotnet ef database update
```

This creates the database schema with User table and indexes.

### 4. Run the Application

```bash
dotnet run
```

The API will start at `https://localhost:<port>`

## API Endpoints

For complete API documentation, including detailed request/response examples, request parameters, and error handling, see **[API Documentation](./docs/API.md)**.

Quick reference:

- `POST /api/auth/register` - Register a new user
- `POST /api/auth/login` - Authenticate user and receive JWT tokens
- `GET /api/auth` - Get authenticated user info (protected)
- `GET /api/auth/admin-only` - Admin-only endpoint (protected)
- `POST /api/auth/refresh-token` - Refresh access token

## Interactive API Documentation

Once the application is running, visit:

```
https://localhost:<port>/scalar/v1
```

This opens an interactive API documentation UI where you can:

- Browse all endpoints
- View request/response schemas
- Test endpoints directly from the browser

## Authentication Flow

### Token Lifecycle

1. **Registration** → User creates account with username & password
2. **Login** → User authenticates, receives `accessToken` (1 hour expiry) and `refreshToken` (7 days expiry)
3. **API Calls** → Include `accessToken` in `Authorization: Bearer {token}` header
4. **Token Expiry** → When `accessToken` expires, use `refreshToken` to get a new one
5. **Refresh Token Expiry** → User must log in again

### JWT Claims

Tokens contain:

- `sub` (Subject) - Username
- `nameid` (Name Identifier) - User ID
- `role` - User role for authorization

## Database Schema

### User Table

```sql
CREATE TABLE [Users] (
    [Id] UNIQUEIDENTIFIER PRIMARY KEY,
    [Username] NVARCHAR(MAX) NOT NULL UNIQUE,
    [PasswordHash] NVARCHAR(MAX) NOT NULL,
    [Role] NVARCHAR(MAX),
    [RefreshToken] NVARCHAR(MAX),
    [RefreshTokenExpiryTime] DATETIME2
);
```

## Key Implementation Details

### Password Security

Passwords are hashed using `PasswordHasher<User>` from ASP.NET Identity:

```csharp
var hashedPassword = new PasswordHasher<User>()
    .HashPassword(user, request.Password);
```

This uses PBKDF2 with SHA-256 by default (configurable).

### JWT Creation

Tokens are signed with HMAC-SHA512:

```csharp
var key = new SymmetricSecurityKey(
    Encoding.UTF8.GetBytes(configuration["AppSettings:Token"]!));
var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha512);
```

### Refresh Token Security

- Refresh tokens are random 32-byte values encoded in Base64
- Stored in database and validated on each refresh
- Automatic expiry after 7 days

## Development

### Build the Project

```bash
dotnet build
```

### Run Tests

```bash
dotnet test
```

### Watch Mode (Auto-reload)

```bash
dotnet watch run
```

## Configuration

### Environment-Specific Settings

- `appsettings.json` - Default settings
- `appsettings.Development.json` - Development overrides
- Environment variables - Take highest precedence

### Key Settings

| Setting | Description |
|---------|-------------|
| `ConnectionStrings:UserDatabase` | SQL Server connection string |
| `AppSettings:Token` | Secret key for JWT signing |
| `AppSettings:Issuer` | JWT issuer claim |
| `AppSettings:Audience` | JWT audience claim |

## Troubleshooting

### Database Connection Failed

```
Verify SQL Server is running:
- Check connection string in appsettings.json
- Ensure database server is accessible
- Check firewall rules
```

### JWT Validation Fails

- Ensure `Token` secret key is the same on client and server
- Verify token hasn't expired
- Check `Issuer` and `Audience` match configuration

### Migration Issues

```bash
# View pending migrations
dotnet ef migrations list

# Rollback last migration
dotnet ef database update <PreviousMigrationName>
```

## Production Deployment

### Security Checklist

- [ ] Use strong, unique JWT secret keys
- [ ] Enable HTTPS only
- [ ] Store secrets in Azure Key Vault or AWS Secrets Manager
- [ ] Implement rate limiting
- [ ] Add CORS policies
- [ ] Enable logging and monitoring
- [ ] Use SQL Server Always Encrypted for sensitive data
- [ ] Implement proper exception handling
- [ ] Add request validation

### Docker Deployment

Create a `Dockerfile`:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["JwtAuthDotNet9.csproj", "."]
RUN dotnet restore "JwtAuthDotNet9.csproj"
COPY . .
RUN dotnet build "JwtAuthDotNet9.csproj" -c Release -o /app/build

FROM mcr.microsoft.com/dotnet/aspnet:9.0
WORKDIR /app
COPY --from=build /app/build .
EXPOSE 80
ENTRYPOINT ["dotnet", "JwtAuthDotNet9.dll"]
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Resources

- [Microsoft JWT Documentation](https://learn.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.jwt)
- [ASP.NET Core Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core)
- [JWT.io](https://jwt.io) - JWT Debugger & Documentation

## Support

For issues, questions, or suggestions, please open an issue or contact the development team.

## 🤖 AI Disclaimer

This README was generated with the assistance of AI and reviewed by a human for clarity and accuracy. ❤️

**Last Updated:** November 2025  
**Framework:** .NET 9  
**License:** MIT
