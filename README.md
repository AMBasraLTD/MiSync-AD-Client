# MiSync Active Directory Client

A modern Windows desktop application for synchronizing users between the SyncPlatform API and on-premises Active Directory.

## Overview

### What is MiSync AD Client?

MiSync AD Client is a Windows service application that automatically synchronizes user accounts from your Management Information System (MIS) to your on-premises Active Directory infrastructure. It acts as a bridge between the cloud-based SyncPlatform API and your local Active Directory environment, ensuring that student and staff accounts are always up-to-date.

### Who is it for?

This application is designed for educational institutions (schools, colleges, academies) that:
- Use a cloud-based MIS system integrated with SyncPlatform
- Maintain an on-premises Active Directory infrastructure for authentication and resource access
- Need automated, reliable synchronization of user accounts between these systems
- Want to maintain organizational structure (year groups, classes, departments) in Active Directory

### Core Capabilities

**Automated User Management**
- Automatically creates new user accounts in Active Directory when users are added to your MIS
- Updates existing user details when changes are made in the MIS
- Disables accounts when users leave or are marked inactive
- Re-enables accounts when users return
- Moves users between organizational units when their role or class changes

**Organizational Structure**
- Automatically creates and maintains a hierarchical OU (Organizational Unit) structure
- Organizes staff by department (Teaching, Admin, Support, SLT)
- Organizes students by year group and registration class
- Dynamically creates new OUs as needed (e.g., new year groups, new classes)
- Maintains a dedicated OU for disabled user accounts

**Username & Password Management**
- Generates usernames according to configurable formats and policies
- Handles duplicate username scenarios intelligently
- Creates secure passwords based on defined policies (different for staff and students)
- Supports password format options including date-of-birth-based and secure random generation

**Safety & Control**
- **Dry Run Mode**: Preview all changes before they're applied to Active Directory
- **Username Conflict Resolution**: Interactive handling of duplicate usernames with multiple resolution strategies
- **Rollback Capability**: Automatically rolls back changes if errors occur during synchronization
- **Ownership Validation**: Ensures the application only modifies accounts it created (prevents accidental modification of existing AD accounts)

**Monitoring & Visibility**
- System tray application that runs in the background
- Real-time dashboard showing sync status and statistics
- Comprehensive logging of all operations
- Sync history tracking with detailed statistics (users created, updated, disabled, errors)
- User action logs showing individual account changes

**Flexibility & Configuration**
- Supports multiple Active Directory authentication methods (Windows Integrated or Service Account)
- Configurable sync intervals (automatic background sync every 15 minutes by default)
- Manual sync trigger for on-demand updates
- Flexible username and password generation rules
- Customizable organizational structure

### How It Works

1. **Heartbeat**: The application sends periodic heartbeats to the SyncPlatform API to indicate it's online and ready to sync
2. **Fetch Users**: Retrieves pending user data from the SyncPlatform API, including students, staff, and their organizational information
3. **Plan Changes**: Compares API data with current Active Directory state and determines what actions need to be taken
4. **Preview (Optional)**: In dry run mode, displays planned changes without making modifications
5. **Execute**: Creates, updates, disables, or moves user accounts in Active Directory according to the plan
6. **Report**: Sends sync results back to the SyncPlatform API, including success/failure status and any errors
7. **Log**: Records all actions and outcomes for audit and troubleshooting purposes

### Key Benefits

- **Time Saving**: Eliminates manual user account creation and maintenance
- **Accuracy**: Reduces human error by automating data transfer
- **Consistency**: Ensures AD structure matches your MIS organization
- **Security**: Automatically disables accounts for leavers, reducing security risks
- **Compliance**: Maintains audit trails of all user account changes
- **Scalability**: Handles organizations of any size, from small schools to large multi-academy trusts

## Features

- **Automatic Synchronization**: Background service that syncs users every 15 minutes (configurable)
- **Manual Sync**: Trigger synchronization on-demand
- **Dry Run Mode**: Preview changes before applying them to Active Directory
- **Dynamic OU Structure**: Automatically creates hierarchical OU structure for staff and students
- **Registration Group Support**: Organizes students by year groups and registration classes
- **Flexible Authentication**: Supports Windows Integrated Authentication or Service Account
- **Username Generation**: Multiple username format options with duplicate handling
- **Password Policies**: Different password formats for staff and students
- **System Tray Integration**: Runs in the background with tray icon
- **Comprehensive Logging**: File-based logging with in-memory buffer for UI display
- **Sync History**: Track all synchronization operations with statistics

## Architecture

### Technology Stack

- .NET 8 (Windows Desktop SDK)
- WPF with MVVM pattern
- System.DirectoryServices for AD operations
- HttpClient with Polly for API resilience
- Windows Registry for configuration storage
- DPAPI for credential encryption

### Project Structure

```
MiSync_AD_Client/
├── MiSync.Client/          # WPF Application
│   ├── Views/              # XAML Windows
│   ├── ViewModels/         # MVVM ViewModels
│   ├── Services/           # Application Services
│   └── Resources/          # Icons, Styles
├── MiSync.Core/            # Core Business Logic
│   ├── API/                # API Client and DTOs
│   ├── ActiveDirectory/    # AD Operations
│   ├── Sync/               # Synchronization Engine
│   ├── Configuration/      # Configuration Management
│   ├── Models/             # Domain Models
│   └── Services/           # Core Services
└── MiSync.Tests/           # Unit Tests
```

## Getting Started

### Prerequisites

- Windows 10/11 or Windows Server 2016+
- .NET 8 SDK
- Access to Active Directory (as Domain Admin or with appropriate permissions)
- SyncPlatform API credentials

### Installation

1. Clone the repository
2. Open `MiSync.sln` in Visual Studio 2022 or later
3. Restore NuGet packages
4. Build the solution
5. Run `MiSync.Client`

### First-Time Setup

1. The application will start in the system tray
2. Right-click the tray icon and select "Settings"
3. Configure the following:
   - **API Settings**: API Key, Organization ID, API Endpoint
   - **AD Settings**: Server, Base DN, Authentication mode
   - **Organization**: School name (used as root OU)
   - **Sync Settings**: Enable AutoSync, configure interval
   - **Username/Password**: Configure generation formats
4. Test connections using the "Test Connection" buttons
5. Save the configuration
6. Go to Dashboard and click "Dry Run Preview" to see planned changes
7. Click "Sync Now" to perform the first synchronization

## Configuration

Configuration is stored in the Windows Registry under `HKEY_CURRENT_USER\Software\MiSync`.

Sensitive data (API keys, passwords) are encrypted using DPAPI (Data Protection API).

### OU Structure

The application creates the following Active Directory structure:

```
<School Name>/
├── Staff/
│   ├── Teaching/
│   ├── Admin/
│   ├── Support/
│   └── SLT/
├── Students/
│   ├── Year 7/
│   │   ├── 7A/
│   │   ├── 7B/
│   │   └── ...
│   ├── Year 8/
│   │   └── ...
│   └── ...
└── Disabled Users/
```

Year groups and registration classes are created dynamically based on data from the API.

## API Integration

The application integrates with the following SyncPlatform API endpoints:

- `POST /organisations/{org_id}/ad-client/heartbeat/` - Send heartbeat
- `GET /organisations/{org_id}/ad-client/pending-users/` - Get users to sync
- `POST /organisations/{org_id}/ad-client/report-status/` - Report sync results
- `GET /organisations/{org_id}/ad-client/user-groupings/` - Get user statistics
- `GET /organisations/{org_id}/ad-client/year-groups/` - Get year groups
- `GET /organisations/{org_id}/ad-client/classes/` - Get registration groups
- `GET /organisations/{org_id}/ad-client/students/` - Get filtered students

## Username Formats

The application supports multiple username formats:

- **FirstName.LastName**: john.smith
- **FirstLast**: johnsmith
- **FirstInitialLastName**: jsmith
- **FirstNameLastInitial**: johnS
- **LastNameFirstInitial**: smithJ

Duplicate handling strategies:
- **AutoIncrement**: john.smith2, john.smith3
- **UseMiddleName**: john.m.smith
- **AppendIdSuffix**: john.smith.12345
- **Skip**: Skip with warning

## Password Policies

Password formats:
- **BasePlusDob**: BasePassword + DDMMYY (e.g., Welcome010105)
- **SecureRandom**: Generated secure random password
- **CustomPattern**: Custom pattern (future enhancement)

Different policies can be configured for staff and students.

## Logging

Logs are stored in: `%AppData%\MiSync\Logs\`

Log files are rotated when they reach 10MB, and old logs are automatically cleaned up after 30 days.

## Sync History

Sync history is stored in the Windows Registry and can be viewed in the Dashboard.

The application retains the last 100 sync operations.

## Troubleshooting

### Connection Issues

- **API Connection Failed**: Check API Key, Organization ID, and endpoint URL
- **AD Connection Failed**: Verify server address, Base DN, and credentials
- Check firewall rules for outgoing HTTPS and LDAP connections

### Sync Errors

- Check logs in `%AppData%\MiSync\Logs\`
- Review sync history in Dashboard for detailed error messages
- Verify AD permissions (user creation, OU creation)

### OU Creation Issues

- Ensure the service account has permissions to create OUs
- Check that Base DN is correct
- Verify school name doesn't contain invalid characters

## Development

### Building from Source

```bash
git clone <repository-url>
cd MiSync_AD_Client
dotnet restore
dotnet build
```

### Running Tests

```bash
dotnet test
```

### Adding New Features

The application uses dependency injection. Register new services in `App.xaml.cs`:

```csharp
services.AddSingleton<YourNewService>();
```

## Security Considerations

- API keys and passwords are encrypted with DPAPI
- Credentials are never logged
- All API calls use HTTPS
- AD operations use minimum required permissions
- Configuration is user-specific (stored in HKCU)

## License

[Add your license information here]

## Support

For issues, questions, or contributions, please contact [your contact information].

## Version History

### Version 1.0.0
- Initial release
- Automatic and manual synchronization
- Dynamic OU structure creation
- Dry run preview
- System tray integration
- Comprehensive logging and history tracking

