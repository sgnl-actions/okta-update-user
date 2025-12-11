# Okta Update User Action

Update an Okta user's profile information using their login username. This action allows you to modify user attributes such as name, email, department, and custom profile fields.

## Overview

This SGNL action integrates with Okta to update a user's profile information. When executed, the specified user attributes will be updated in Okta.

## Prerequisites

- Okta instance
- API authentication credentials (supports 4 auth methods - see Configuration below)
- Okta API access with permissions to update user profiles

## Configuration

### Authentication

This action supports four authentication methods. Configure one of the following:

#### Option 1: Bearer Token (Okta API Token)
| Secret | Description |
|--------|-------------|
| `BEARER_AUTH_TOKEN` | Okta API token (SSWS format) |

#### Option 2: Basic Authentication
| Secret | Description |
|--------|-------------|
| `BASIC_USERNAME` | Username for Okta authentication |
| `BASIC_PASSWORD` | Password for Okta authentication |

#### Option 3: OAuth2 Client Credentials
| Secret/Environment | Description |
|-------------------|-------------|
| `OAUTH2_CLIENT_CREDENTIALS_CLIENT_SECRET` | OAuth2 client secret |
| `OAUTH2_CLIENT_CREDENTIALS_CLIENT_ID` | OAuth2 client ID |
| `OAUTH2_CLIENT_CREDENTIALS_TOKEN_URL` | OAuth2 token endpoint URL |
| `OAUTH2_CLIENT_CREDENTIALS_SCOPE` | OAuth2 scope (optional) |
| `OAUTH2_CLIENT_CREDENTIALS_AUDIENCE` | OAuth2 audience (optional) |
| `OAUTH2_CLIENT_CREDENTIALS_AUTH_STYLE` | OAuth2 auth style (optional) |

#### Option 4: OAuth2 Authorization Code
| Secret | Description |
|--------|-------------|
| `OAUTH2_AUTHORIZATION_CODE_ACCESS_TOKEN` | OAuth2 access token |

### Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ADDRESS` | Default Okta API base URL | `https://dev-12345.okta.com` |

### Input Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `login` | string | Yes | The Okta user's login username (typically email) | `jane.smith@example.com` |
| `address` | string | No | Override API base URL | `https://custom.okta.com` |
| `firstName` | string | No | User's first name | `Jane` |
| `lastName` | string | No | User's last name | `Smith` |
| `email` | string | No | User's email address | `jane.smith@example.com` |
| `department` | string | No | User's department | `Engineering` |
| `employeeNumber` | string | No | Employee number | `EMP12345` |
| `additionalProfileAttributes` | string | No | JSON string of additional profile attributes | `{"mobilePhone": "555-1234"}` |

**Note**: At least one profile field (firstName, lastName, email, department, employeeNumber, or additionalProfileAttributes) must be provided.

### Output Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | The user ID that was updated |
| `status` | string | User account status |
| `profile` | object | Updated user profile information |
| `created` | datetime | When the user was created (ISO 8601) |
| `activated` | datetime | When the user was activated (ISO 8601) |
| `lastLogin` | datetime | Last login timestamp (ISO 8601) |
| `lastUpdated` | datetime | When the user was last updated (ISO 8601) |

## Usage Example

### Job Request

```json
{
  "id": "update-user-001",
  "type": "nodejs-22",
  "script": {
    "repository": "github.com/sgnl-actions/okta-update-user",
    "version": "v1.0.0",
    "type": "nodejs"
  },
  "script_inputs": {
    "login": "jane.smith@example.com",
    "firstName": "Jane",
    "lastName": "Smith",
    "department": "Engineering"
  },
  "environment": {
    "ADDRESS": "https://dev-12345.okta.com",
    "LOG_LEVEL": "info"
  }
}
```

### Successful Response

```json
{
  "id": "00u1234567890abcdef",
  "status": "ACTIVE",
  "created": "2024-01-01T10:00:00.000Z",
  "activated": "2024-01-01T10:00:00.000Z",
  "lastLogin": "2024-01-15T08:30:00.000Z",
  "lastUpdated": "2024-01-16T14:15:00.000Z",
  "profile": {
    "firstName": "Jane",
    "lastName": "Smith",
    "email": "jane.smith@example.com",
    "login": "jane.smith@example.com",
    "department": "Engineering"
  }
}
```

## How It Works

The action performs a POST request to the Okta API to update the user profile:

1. **Validate Input**: Ensures login is provided and at least one profile field is specified
2. **Build Profile Update**: Constructs the profile update payload with provided fields
3. **Authenticate**: Uses configured authentication method to get authorization
4. **URL Encode Login**: Properly encodes the login (handles special characters like + and @)
5. **Update User**: Makes POST request to `/api/v1/users/{encodedLogin}`
6. **Return Result**: Returns the updated user object

## Error Handling

The action includes error handling for common scenarios:

### HTTP Status Codes
- **200 OK**: Successful update (expected response)
- **400 Bad Request**: Invalid login or profile data format
- **401 Unauthorized**: Invalid authentication credentials
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: User not found
- **429 Rate Limit**: Too many requests

### Common Errors
- **Invalid or missing login parameter**: login must be provided
- **At least one profile field must be provided to update**: No update fields specified
- **Invalid additionalProfileAttributes JSON**: JSON parsing failed

## Development

### Local Testing

```bash
# Install dependencies
npm install

# Run tests
npm test

# Test locally with mock data
npm run dev

# Build for production
npm run build
```

### Running Tests

The action includes comprehensive unit tests covering:
- Input validation (login parameter, profile fields)
- Authentication handling (all 4 auth methods)
- Success scenarios (single and multiple field updates)
- Error handling (API errors, missing credentials, invalid JSON)
- Empty string field handling
- Special character encoding in login (e.g., test+special@example.com)

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Check test coverage
npm run test:coverage
```

## Security Considerations

- **Credential Protection**: Never log or expose authentication credentials
- **User Impact**: Profile changes are immediate and visible to applications
- **Audit Logging**: All operations are logged with timestamps
- **Input Validation**: Login and profile data are validated
- **URL Encoding**: Login values are properly URL-encoded to handle special characters
- **Data Integrity**: Only specified fields are updated; other profile data remains unchanged

## Okta API Reference

This action uses the following Okta API endpoint:
- [Update User](https://developer.okta.com/docs/reference/api/users/#update-user) - POST `/api/v1/users/{login}`

## Troubleshooting

### Common Issues

1. **"Invalid or missing login parameter"**
   - Ensure the `login` parameter is provided and is a non-empty string
   - Verify the login exists in your Okta instance

2. **"At least one profile field must be provided to update"**
   - Provide at least one of: firstName, lastName, email, department, employeeNumber, or additionalProfileAttributes
   - Empty strings are ignored; use actual values

3. **"Invalid additionalProfileAttributes JSON"**
   - Ensure additionalProfileAttributes is valid JSON
   - Example: `{"mobilePhone": "555-1234", "title": "Engineer"}`

4. **"No URL specified. Provide address parameter or ADDRESS environment variable"**
   - Set the ADDRESS environment variable or provide address parameter
   - Example: `https://dev-12345.okta.com`

5. **"No authentication configured"**
   - Ensure you have configured one of the four supported authentication methods
   - Check that the required secrets/environment variables are set

6. **"Failed to update user: HTTP 404"**
   - Verify the login is correct
   - Check that the user exists in Okta

7. **"Failed to update user: HTTP 403"**
   - Ensure your API credentials have permission to update user profiles
   - Check Okta admin console for required permissions

## Version History

### v1.0.0
- Initial release
- Support for updating users by login via Okta API
- Four authentication methods (Bearer, Basic, OAuth2 Client Credentials, OAuth2 Authorization Code)
- Support for standard and custom profile attributes
- Proper URL encoding for login values with special characters
- Integration with @sgnl-actions/utils package
- Comprehensive error handling

## License

MIT

## Support

For issues or questions, please contact SGNL Engineering or create an issue in this repository.
