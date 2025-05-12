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
2. Download the latest .NET SDK (version 8)
3. Follow the installation instructions for your operating system
4. Update and verify the installation by opening a terminal/command prompt and running:
   ```sh
   dotnet tool install --global dotnet-ef --framework net8.0
   dotnet --version
   ```
   This should display the installed .NET version

## Installing C# Dev Kit Extension

1. Open Visual Studio Code
2. Click on the Extensions icon in the sidebar (or press Ctrl+Shift+X)
3. Search for "C# Dev Kit"
4. Click "Install"
5. Search for "IntelliCode for C# Dev KitPreview"
6. Click "Install"
7. You may need to restart VS Code after installation

## Installing PostgreSQL

### Windows
1. Visit the [PostgreSQL download for Windows OS page](https://www.postgresql.org/download/windows/)
2. Download the version 16.9 installer from EnterpriseDB
3. Run the installer and follow the prompts
4. Remember the password you set for the postgres user
5. Keep the default port (5432)
6. Complete the installation

### macOS
1. The easiest way to install PostgreSQL on macOS is using Homebrew. Check if you have it installed by running the following command in the terminal.
   ```sh
   which brew
   ```
2. If you don't have Homebrew installed, install it first:
   ```sh
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

## Troubleshooting

If you encounter issues during installation:

- **.NET SDK**: Visit the [.NET troubleshooting guide](https://docs.microsoft.com/en-us/dotnet/core/install/troubleshoot)
- **PostgreSQL**: Check the [PostgreSQL documentation](https://www.postgresql.org/docs/)
- **VS Code Extensions**: Try uninstalling and reinstalling the extension

## Next Steps

Once you have all the required software installed, you're ready to begin the first project in Book 1. Start with the [Setting up a console app](./setting-up-console-app.md) chapter in the C# Quick Intro column.