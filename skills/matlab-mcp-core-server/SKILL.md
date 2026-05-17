---
name: matlab-mcp-core-server
description: Run MATLAB from AI applications using the official MathWorks MCP server to execute code, analyze scripts, and manage MATLAB sessions.
triggers:
  - run MATLAB code from my AI assistant
  - execute this MATLAB script
  - analyze my MATLAB code for issues
  - connect to a running MATLAB session
  - check what MATLAB toolboxes are installed
  - set up MATLAB MCP server
  - evaluate MATLAB commands
  - start MATLAB in desktop mode
---

# MATLAB MCP Core Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The MATLAB MCP Core Server enables AI applications to interact with MATLAB, allowing you to write and execute MATLAB code, perform static code analysis, and manage MATLAB sessions directly from your AI coding assistant.

## What It Does

- **Start and control MATLAB** from AI applications (Claude Code, VS Code Copilot, etc.)
- **Execute MATLAB code** and scripts, returning output to your AI assistant
- **Analyze MATLAB code** for style issues, potential errors, and best practices
- **Detect installed toolboxes** and MATLAB version information
- **Connect to existing MATLAB sessions** or start new ones
- **Support multiple display modes** (desktop or headless)

## Installation

### Prerequisites

1. **Install MATLAB** R2021a or later and add it to your system PATH
2. **Download the server binary** from [releases](https://github.com/matlab/matlab-mcp-core-server/releases/latest)

### For Claude Code

```bash
# Add the server (replace with your actual binary path)
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-core-server

# With custom working folder
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-core-server --initial-working-folder=/home/user/myproject

# With nodesktop mode
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-core-server --matlab-display-mode=nodesktop
```

### For VS Code (GitHub Copilot)

Create `.vscode/mcp.json` in your workspace:

```json
{
    "servers": {
        "matlab": {
            "type": "stdio",
            "command": "/path/to/matlab-mcp-core-server",
            "args": [
                "--initial-working-folder=/home/user/projects",
                "--matlab-display-mode=nodesktop"
            ]
        }
    }
}
```

Windows example:

```json
{
    "servers": {
        "matlab": {
            "type": "stdio",
            "command": "C:\\Program Files\\matlab-mcp-core-server\\matlab-mcp-core-server-win64.exe",
            "args": [
                "--initial-working-folder=C:\\Users\\username\\projects",
                "--initialize-matlab-on-startup=true"
            ]
        }
    }
}
```

### For Claude Desktop

1. Download `matlab-mcp-core-server.mcpb` from releases
2. Install Filesystem extension in Claude Desktop
3. Double-click the `.mcpb` file and click Install
4. Configure via Settings > Extensions > Configure

## Configuration Arguments

All arguments can be specified as CLI flags, in your MCP config, or as environment variables (prefix `MW_MCP_SERVER_`, uppercase, underscores).

### Essential Arguments

```bash
# Specify MATLAB installation
--matlab-root=/usr/local/MATLAB/R2026a
# Environment variable: MW_MCP_SERVER_MATLAB_ROOT

# Set initial working directory
--initial-working-folder=/home/user/myproject
# Environment variable: MW_MCP_SERVER_INITIAL_WORKING_FOLDER

# Display mode: desktop or nodesktop
--matlab-display-mode=nodesktop
# Environment variable: MW_MCP_SERVER_MATLAB_DISPLAY_MODE

# Initialize MATLAB immediately on startup
--initialize-matlab-on-startup=true
# Environment variable: MW_MCP_SERVER_INITIALIZE_MATLAB_ON_STARTUP
```

### Session Management

```bash
# Connect to existing MATLAB session (R2023a+)
--matlab-session-mode=existing
# Environment variable: MW_MCP_SERVER_MATLAB_SESSION_MODE

# Setup for existing session mode (run once)
./matlab-mcp-core-server --setup-matlab
```

### Advanced Arguments

```bash
# Custom tools definition
--extension-file=/path/to/my-tools.json
# Environment variable: MW_MCP_SERVER_EXTENSION_FILE

# Log configuration
--log-folder=/tmp/matlab-mcp-logs
--log-level=debug
# Environment variables: MW_MCP_SERVER_LOG_FOLDER, MW_MCP_SERVER_LOG_LEVEL

# Disable telemetry
--disable-telemetry=true
# Environment variable: MW_MCP_SERVER_DISABLE_TELEMETRY
```

## Available Tools

### 1. detect_matlab_toolboxes

Returns information about installed MATLAB version and toolboxes.

**Usage in AI conversation:**
- "What MATLAB toolboxes are installed?"
- "Check my MATLAB version"
- "List available MATLAB products"

**Example response:**
```
MATLAB Version: R2026a
Installed Toolboxes:
- Signal Processing Toolbox (9.3)
- Image Processing Toolbox (12.0)
- Deep Learning Toolbox (24.1)
```

### 2. check_matlab_code

Performs static code analysis without executing the script.

**Parameters:**
- `script_path` (string): Absolute path to `.m` file

**Usage in AI conversation:**
- "Analyze this MATLAB script for issues"
- "Check my code quality in analysis.m"
- "What coding style problems are in /home/user/matlab/process_data.m?"

**Example:**
```
Script: /home/user/projects/calculate_fft.m

Analysis results:
- Line 15: Variable 'N' is defined but never used
- Line 23: Use of deprecated function 'fft2'. Consider 'fft' instead
- Line 45: Missing semicolon may cause unwanted output
- Performance: Consider preallocating array at line 12
```

### 3. evaluate_matlab_code

Executes MATLAB code string and returns output.

**Parameters:**
- `code` (string): MATLAB code to execute
- `project_path` (string): Working directory for execution

**Usage in AI conversation:**
- "Run this MATLAB code: `x = 1:10; mean(x)`"
- "Execute MATLAB: `plot(sin(0:0.1:2*pi))`"
- "Calculate in MATLAB: `A = magic(5); det(A)`"

**Example code snippets:**

Simple calculation:
```matlab
x = linspace(0, 2*pi, 100);
y = sin(x);
plot(x, y);
title('Sine Wave');
```

Matrix operations:
```matlab
A = [1 2 3; 4 5 6; 7 8 9];
B = inv(A' * A) * A';
disp('Moore-Penrose pseudoinverse:');
disp(B);
```

Signal processing:
```matlab
fs = 1000;
t = 0:1/fs:1-1/fs;
signal = sin(2*pi*50*t) + 0.5*sin(2*pi*120*t);
fft_result = fft(signal);
power = abs(fft_result).^2/length(signal);
fprintf('Signal power: %.4f\n', sum(power));
```

### 4. run_matlab_file

Executes a MATLAB script file.

**Parameters:**
- `script_path` (string): Absolute path to `.m` file

**Usage in AI conversation:**
- "Run the MATLAB script at /home/user/analysis.m"
- "Execute my data processing script"
- "Run C:\Users\name\projects\simulation.m"

**Example script (`process_data.m`):**
```matlab
% Load and process experimental data
data = load('experiment_results.mat');
temperature = data.temperature;
pressure = data.pressure;

% Remove outliers
temp_clean = rmoutliers(temperature);
press_clean = rmoutliers(pressure);

% Calculate statistics
mean_temp = mean(temp_clean);
std_temp = std(temp_clean);
mean_press = mean(press_clean);

% Create visualization
figure;
subplot(2,1,1);
histogram(temp_clean, 30);
title(sprintf('Temperature Distribution (μ=%.2f, σ=%.2f)', mean_temp, std_temp));
xlabel('Temperature (°C)');

subplot(2,1,2);
scatter(temp_clean, press_clean);
title('Pressure vs Temperature');
xlabel('Temperature (°C)');
ylabel('Pressure (kPa)');

% Save results
results.mean_temperature = mean_temp;
results.mean_pressure = mean_press;
results.correlation = corrcoef(temp_clean, press_clean);
save('processed_results.mat', 'results');

fprintf('Processing complete. Results saved.\n');
```

## Common Patterns

### Interactive Data Analysis

**AI conversation flow:**
```
User: "Load the data from sensor_readings.csv and plot it"

AI uses: evaluate_matlab_code
Code: 
data = readtable('sensor_readings.csv');
plot(data.Time, data.Value);
xlabel('Time'); ylabel('Sensor Reading');
grid on;
```

### Code Quality Workflow

**AI conversation flow:**
```
User: "Check my simulation script for issues before running it"

AI uses: check_matlab_code (script_path: /home/user/simulation.m)
Then: Suggests fixes
Then: evaluate_matlab_code (to run corrected version)
```

### Multi-Step Analysis

```matlab
% Step 1: Load and prepare data
data = readmatrix('measurements.csv');
x = data(:,1);
y = data(:,2);

% Step 2: Fit polynomial model
p = polyfit(x, y, 3);
y_fit = polyval(p, x);

% Step 3: Calculate residuals and R-squared
residuals = y - y_fit;
ss_res = sum(residuals.^2);
ss_tot = sum((y - mean(y)).^2);
r_squared = 1 - (ss_res / ss_tot);

% Step 4: Visualize results
figure;
plot(x, y, 'o', 'DisplayName', 'Data');
hold on;
plot(x, y_fit, '-', 'LineWidth', 2, 'DisplayName', 'Fit');
legend('show');
title(sprintf('Polynomial Fit (R² = %.4f)', r_squared));
```

### Connecting to Existing MATLAB Session

**Setup (one-time):**
```bash
# Install the MCP toolbox in MATLAB
./matlab-mcp-core-server --setup-matlab --matlab-root=/usr/local/MATLAB/R2026a
```

**In running MATLAB session:**
```matlab
% Share this MATLAB session with MCP
shareMATLABSession()
```

**Configure MCP server:**
```json
{
    "servers": {
        "matlab": {
            "type": "stdio",
            "command": "/path/to/matlab-mcp-core-server",
            "args": ["--matlab-session-mode=existing"]
        }
    }
}
```

## Troubleshooting

### MATLAB Not Found

**Error:** Server cannot locate MATLAB installation

**Solution:**
```bash
# Explicitly specify MATLAB root
--matlab-root=/usr/local/MATLAB/R2026a

# Or add MATLAB to PATH
export PATH="/usr/local/MATLAB/R2026a/bin:$PATH"
```

### Permission Denied on macOS

**Error:** Cannot execute binary on macOS

**Solution:**
```bash
chmod +x ~/Downloads/matlab-mcp-core-server
```

### Graphics Issues in nodesktop Mode

**Problem:** Plots don't appear in `nodesktop` mode

**Solution:** Use `desktop` mode or save figures to files:
```matlab
figure;
plot(x, y);
saveas(gcf, 'output.png');
```

### Script Execution Errors

**Error:** "File not found" when running scripts

**Solution:** Use absolute paths and verify file location:
```bash
# Correct
script_path: /home/user/projects/analysis.m

# Incorrect (relative paths may fail)
script_path: ../analysis.m
```

### Existing Session Not Connecting

**Problem:** Server doesn't connect to running MATLAB

**Solution:**
1. Verify MATLAB version is R2023a or later
2. Run `shareMATLABSession()` in MATLAB command window
3. Ensure only one MATLAB session is shared
4. Check that `--setup-matlab` was run successfully

### Memory Issues with Large Data

**Problem:** MATLAB runs out of memory

**Solution:**
```matlab
% Process data in chunks
chunk_size = 1000;
for i = 1:chunk_size:length(data)
    chunk = data(i:min(i+chunk_size-1, end));
    process_chunk(chunk);
end

% Clear variables when done
clear large_array;
```

### Log File Location

**Find logs for debugging:**
```bash
# Specify custom log location
--log-folder=/home/user/matlab-logs --log-level=debug

# Default locations:
# Linux/macOS: /tmp
# Windows: C:\Users\username\AppData\Local\Temp
```

### Code Analysis False Positives

**Problem:** `check_matlab_code` reports valid code as problematic

**Note:** Static analysis may flag intentional patterns. Use `evaluate_matlab_code` to verify code actually works as intended.

## Environment Variables Reference

```bash
# Core configuration
export MW_MCP_SERVER_MATLAB_ROOT="/usr/local/MATLAB/R2026a"
export MW_MCP_SERVER_INITIAL_WORKING_FOLDER="/home/user/projects"
export MW_MCP_SERVER_MATLAB_DISPLAY_MODE="nodesktop"
export MW_MCP_SERVER_MATLAB_SESSION_MODE="existing"
export MW_MCP_SERVER_INITIALIZE_MATLAB_ON_STARTUP="true"

# Advanced options
export MW_MCP_SERVER_EXTENSION_FILE="/path/to/tools.json"
export MW_MCP_SERVER_LOG_FOLDER="/var/log/matlab-mcp"
export MW_MCP_SERVER_LOG_LEVEL="info"
export MW_MCP_SERVER_DISABLE_TELEMETRY="true"
```

## Best Practices

1. **Use absolute paths** for all file references
2. **Specify working directories** via `project_path` or `initial-working-folder`
3. **Check code before running** with `check_matlab_code` for complex scripts
4. **Use nodesktop mode** for production/server environments
5. **Connect to existing sessions** when iterating on live data analysis
6. **Save figures to files** when working in headless mode
7. **Clear large variables** explicitly to manage memory
8. **Use try-catch blocks** in scripts to handle errors gracefully
