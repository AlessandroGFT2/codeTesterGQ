# Code Tester GA - Automated Unit Test Generator

This project automatically generates unit tests for your codebase using AI-powered test generation. The workflow consists of three sequential scripts that scan your code, send it for AI processing, and download the generated tests.

## 📋 Prerequisites

### Required Software
- **Node.js** (version 18 or higher)
- **Git** (for GitHub Actions workflow)
- **Access to GFT AI Impact API** (internal network required)

### Required Environment Variables
Create a `.env` file in the root directory:
```env
ACCESS_TOKEN=your_keycloak_access_token_here
LOCAL_MACHINE=true
```

## 🚀 Quick Start

### 1. Configure the Project

Edit `codeTesterGA/config.json` with your project settings:

```json
{
    "foldersToInclude": [
        "./src",
        "./lib"
    ],
    "foldersToExclude": [
        "node_modules",
        "dist",
        "build"
    ],
    "extensionsIncluded": [
        ".js",
        ".java",
        ".ts",
        ".py"
    ],
    "testFrameworks": {
        ".js": "jest",
        ".java": "junit",
        ".ts": "jest",
        ".py": "pytest"
    },
    "testsFolder": "./generatedTests",
    "promptId": "TestCreator__CreateUnitTests_V1",
    "additionalInstructions": "Generate only the source code, without any extra information",
    "llm": "AWS_CLAUDE",
    "extensionToLanguage": {
        ".js": "JavaScript",
        ".ts": "TypeScript",
        ".java": "Java",
        ".py": "Python"
    }
}
```

### 2. Install Dependencies

```bash
npm install axios form-data dotenv
```

### 3. Run the Scripts

Execute the scripts in order:

```bash
# Step 1: Generate files list
node codeTesterGA/generate-files-list.js

# Step 2: Send files to AI for processing
node codeTesterGA/send-files.js

# Step 3: Download generated tests
node codeTesterGA/download-files.js
```

## 📖 Configuration Guide

### Required Configuration Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `foldersToInclude` | Array | Directories to scan for source files | `["./src", "./lib"]` |
| `foldersToExclude` | Array | Directories to ignore (can be empty) | `["node_modules", "test"]` |
| `extensionsIncluded` | Array | File extensions to process | `[".js", ".java", ".ts"]` |
| `testFrameworks` | Object | Test framework for each extension | `{".js": "jest"}` |
| `testsFolder` | String | Output directory for generated tests | `"./generatedTests"` |
| `promptId` | String | AI prompt template ID | `"TestCreator__CreateUnitTests_V1"` |
| `additionalInstructions` | String | Custom instructions for AI | `"Generate comprehensive tests"` |
| `llm` | String | AI model to use | `"AWS_CLAUDE"` |

### Path Configuration

**✅ Use relative paths** for cross-platform compatibility:
```json
{
    "foldersToInclude": ["./src", "./components"],
    "testsFolder": "./generatedTests"
}
```

**❌ Avoid absolute paths** (won't work in GitHub Actions):
```json
{
    "foldersToInclude": ["C:/Users/user/project/src"],
    "testsFolder": "C:/Users/user/project/tests"
}
```

## 🔄 Script Workflow

### 1. `generate-files-list.js`
- **Purpose**: Scans configured directories for source files
- **Input**: Configuration from `config.json`
- **Output**: Creates `codeTesterFiles.json` with file metadata
- **Behavior**: Exits if `codeTesterFiles.json` already exists

**Sample output:**
```json
[
  {
    "fileName": "UserService.js",
    "originalPath": "src/services/UserService.js",
    "jobId": null,
    "error": null,
    "uri": null,
    "downloaded": false
  }
]
```

### 2. `send-files.js`
- **Purpose**: Sends source files to AI API for test generation
- **Input**: Files from `codeTesterFiles.json`
- **Process**: 
  - Reads each source file
  - Sends to AI API with configuration
  - Updates `codeTesterFiles.json` with job IDs
- **Rate Limiting**: 250ms delay between requests

### 3. `download-files.js`
- **Purpose**: Downloads generated tests when AI processing completes
- **Process**:
  - Polls job status every 500ms
  - Downloads completed tests
  - Saves to configured `testsFolder`
- **Output**: Generated test files in directory structure

## 🤖 GitHub Actions Integration

### Workflow Setup

1. **Move workflow file** to `.github/workflows/`:
```bash
mkdir -p .github/workflows
mv codeTesterGA/codeTesterWorkflow.yml .github/workflows/
```

2. **Set up GitHub secrets**:
   - `PAT_GITHUB_TOKEN`: Personal Access Token for PR creation

3. **Trigger workflow**:
   - Manual: Go to Actions tab → "CI Integrado" → "Run workflow"
   - Automatic: Configure triggers in the workflow file

### Workflow Process

1. **Authentication**: Gets token from Keycloak
2. **Dependency Installation**: Installs required npm packages
3. **File Processing**: Runs all three scripts sequentially
4. **PR Creation**: Creates pull request with generated tests

## 📁 Output Structure

Generated tests maintain your source directory structure:

```
generatedTests/
├── src/
│   ├── services/
│   │   └── UserService.test.js
│   └── components/
│       └── Login.test.java
└── lib/
    └── utils/
        └── helpers.test.ts
```

## 🔧 Troubleshooting

### Common Issues

**Script does nothing / no logging:**
- Check if `codeTesterFiles.json` has `"downloaded": true` for all files
- Verify `jobId` is not null
- Ensure no error messages in the JSON file

**Connection errors:**
```
Error sending file: connect ECONNREFUSED
```
- Verify VPN/network access to GFT AI Impact API
- Check ACCESS_TOKEN is valid and not expired

**File path issues:**
- Use relative paths in configuration
- Ensure working directory is project root when running scripts

**Module warnings:**
```bash
# Add to package.json to fix ES module warnings
echo '{"type": "module"}' > package.json
```

### Reset and Restart

To start fresh:
```bash
# Delete generated files
rm codeTesterGA/codeTesterFiles.json
rm -rf generatedTests/

# Run workflow again
node codeTesterGA/generate-files-list.js
```

## 📋 Supported Languages & Frameworks

| Language | Extensions | Default Framework |
|----------|------------|------------------|
| JavaScript | `.js`, `.mjs`, `.cjs` | Jest |
| TypeScript | `.ts`, `.tsx` | Jest |
| Java | `.java` | JUnit |
| Python | `.py`, `.pyw` | PyTest |
| C# | `.cs` | NUnit |
| Go | `.go` | Go Test |

## 🛡️ Security Notes

- **Never commit** `.env` files with tokens
- **Use GitHub secrets** for CI/CD environments
- **Tokens expire** - refresh Keycloak tokens regularly
- **Network access** required to GFT AI Impact API

## 📞 Support

For issues with:
- **Script errors**: Check configuration and file paths
- **API access**: Contact GFT AI Impact team
- **GitHub Actions**: Verify secrets and workflow permissions