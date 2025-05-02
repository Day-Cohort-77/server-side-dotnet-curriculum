# Installations for Book 1

To complete the exercises and projects in Book 1, you'll need to install several tools and software packages. This guide will walk you through the installation process for each required component.

## Required Software

1. **.NET SDK** - For C# development
2. **Visual Studio Code** - Code editor
3. **C# Dev Kit Extension** - For C# development in VS Code
4. **PostgreSQL** - Database system
5. **pgAdmin** - PostgreSQL management tool
6. **Postman** - API testing tool

## Installing .NET SDK

1. Visit the [.NET download page](https://dotnet.microsoft.com/download)
2. Download the latest .NET SDK (version 8.0 or later)
3. Follow the installation instructions for your operating system
4. Verify the installation by opening a terminal/command prompt and running:
   ```
   dotnet --version
   ```
   This should display the installed .NET version

## Installing Visual Studio Code

1. Visit the [Visual Studio Code download page](https://code.visualstudio.com/download)
2. Download the appropriate version for your operating system
3. Follow the installation instructions
4. Launch VS Code after installation

## Installing C# Dev Kit Extension

1. Open Visual Studio Code
2. Click on the Extensions icon in the sidebar (or press Ctrl+Shift+X)
3. Search for "C# Dev Kit"
4. Click "Install"
5. You may need to restart VS Code after installation

## Installing PostgreSQL

### Windows
1. Visit the [PostgreSQL download page](https://www.postgresql.org/download/windows/)
2. Download the installer from EnterpriseDB
3. Run the installer and follow the prompts
4. Remember the password you set for the postgres user
5. Keep the default port (5432)
6. Complete the installation

### macOS
1. The easiest way to install PostgreSQL on macOS is using Homebrew
2. If you don't have Homebrew installed, install it first:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
3. Install PostgreSQL:
   ```
   brew install postgresql
   ```
4. Start the PostgreSQL service:
   ```
   brew services start postgresql
   ```

### Linux
1. For Ubuntu/Debian:
   ```
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   ```
2. For Fedora:
   ```
   sudo dnf install postgresql-server postgresql-contrib
   sudo systemctl enable postgresql
   sudo postgresql-setup --initdb --unit postgresql
   sudo systemctl start postgresql
   ```

## Installing pgAdmin

1. Visit the [pgAdmin download page](https://www.pgadmin.org/download/)
2. Select your operating system
3. Download and install the latest version
4. Launch pgAdmin after installation
5. You'll be prompted to set a master password for pgAdmin
6. Connect to your PostgreSQL server using the credentials you set during PostgreSQL installation

## Installing Postman

1. Visit the [Postman download page](https://www.postman.com/downloads/)
2. Download the appropriate version for your operating system
3. Install and launch Postman
4. You can create a free account or skip sign-in for now

## Verifying Your Installations

To ensure everything is installed correctly:

1. Open a terminal/command prompt
2. Run the following commands:
   ```
   dotnet --version
   psql --version
   ```
3. Both commands should return version information
4. Open VS Code and ensure the C# Dev Kit extension is active
5. Open pgAdmin and verify you can connect to your PostgreSQL server
6. Open Postman and verify it launches correctly

## Troubleshooting

If you encounter issues during installation:

- **.NET SDK**: Visit the [.NET troubleshooting guide](https://docs.microsoft.com/en-us/dotnet/core/install/troubleshoot)
- **PostgreSQL**: Check the [PostgreSQL documentation](https://www.postgresql.org/docs/)
- **VS Code Extensions**: Try uninstalling and reinstalling the extension

## Next Steps

Once you have all the required software installed, you're ready to begin the first project in Book 1. Start with the [Setting up a console app](./setting-up-console-app.md) chapter in the C# Quick Intro column.