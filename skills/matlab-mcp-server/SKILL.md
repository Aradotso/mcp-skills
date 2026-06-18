---
name: matlab-mcp-server
description: Run and interact with MATLAB using AI applications through the Model Context Protocol, enabling AI agents to execute MATLAB code, manage sessions, and assess code quality.
triggers:
  - run this matlab code
  - execute matlab script
  - start matlab session
  - analyze matlab code quality
  - evaluate matlab expression
  - connect to existing matlab
  - check matlab code style
  - run matlab calculations
---

# MATLAB MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The MATLAB MCP Server is the official MathWorks server that enables AI applications to start MATLAB, execute MATLAB code, and assess code quality through the Model Context Protocol. It supports multiple session modes, custom working directories, and both desktop and headless MATLAB operation.

## Installation

### Prerequisites

- MATLAB R2021a or later installed and added to system PATH
- The server supports MATLAB releases from the past five years

### For Claude Code

```bash
# Basic installation
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-server

# With custom working folder
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-server --initial-working-folder=/home/user/project

# With nodesktop mode
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-server --matlab-display-mode=nodesktop
```

### For Claude Desktop

1. Install the Filesystem extension in Claude Desktop (Settings > Extensions > Browse extensions)
2. Download `matlab-mcp-server.mcpb` from the [latest release](https://github.com/matlab/matlab-mcp-server/releases/latest)
3. Double-click the `.mcpb` file and click Install

### For VS Code with GitHub Copilot

Create `.vscode/mcp.json`:

```json
{
    "servers": {
        "matlab": {
            "type": "stdio",
            "command": "/path/to/matlab-mcp-server",
            "args": [
                "--initial-working-folder=/home/user/project",
                "--matlab-display-mode=nodesktop"
            ]
        }
    }
}
```

### Download Binary

**Linux/macOS:**
```bash
# macOS Apple Silicon
curl -L -o ~/Downloads/matlab-mcp-server https://github.com/matlab/matlab-mcp-server/releases/latest/download/matlab-mcp-server-macos-amd64
chmod +x ~/Downloads/matlab-mcp-server

# macOS Intel
curl -L -o ~/Downloads/matlab-mcp-server https://github.com/matlab/matlab-mcp-server/releases/latest/download/matlab-mcp-server-macos-x64
chmod +x ~/Downloads/matlab-mcp-server
```

**Windows:**
Download from [releases page](https://github.com/matlab/matlab-mcp-server/releases/latest): `matlab-mcp-server-windows-x64.exe`

**Build from source:**
```bash
go install github.com/matlab/matlab-mcp-server/cmd/matlab-mcp-server@latest
```

## Configuration Arguments

### Command-Line Flags

```bash
# Specify MATLAB installation
--matlab-root=/usr/local/MATLAB/R2026a

# Initialize MATLAB immediately on startup
--initialize-matlab-on-startup=true

# Set working directory
--initial-working-folder=/home/user/myproject

# Run without MATLAB desktop
--matlab-display-mode=nodesktop

# Session modes
--matlab-session-mode=new        # Always start new MATLAB
--matlab-session-mode=auto       # Connect to existing or start new (default)
--matlab-session-mode=existing   # Only connect to existing MATLAB
```

### Environment Variables

```bash
# Equivalent to --matlab-root
export MW_MCP_SERVER_MATLAB_ROOT=/usr/local/MATLAB/R2026a

# Equivalent to --initial-working-folder
export MW_MCP_SERVER_INITIAL_WORKING_FOLDER=/home/user/project

# Equivalent to --matlab-display-mode
export MW_MCP_SERVER_MATLAB_DISPLAY_MODE=nodesktop

# Equivalent to --matlab-session-mode
export MW_MCP_SERVER_MATLAB_SESSION_MODE=existing
```

### Using Existing MATLAB Sessions

For MATLAB R2023a and later:

1. **First-time setup:**
```bash
./matlab-mcp-server --setup-matlab
```
This installs the MATLAB MCP Server Toolbox.

2. **In MATLAB command window:**
```matlab
shareMATLABSession()
```
Add this to your MATLAB `startup.m` for automatic sharing:
```matlab
% In startup.m
shareMATLABSession()
```

3. **Configure MCP server:**
```bash
claude mcp add --transport stdio matlab -- /path/to/matlab-mcp-server --matlab-session-mode=existing
```

## MCP Tools Available

The server exposes these tools to AI applications:

### execute_matlab_code

Execute MATLAB code and return results.

**Request:**
```json
{
  "name": "execute_matlab_code",
  "arguments": {
    "code": "result = sum([1, 2, 3, 4, 5]); disp(result)"
  }
}
```

**Response:**
```json
{
  "content": [
    {
      "type": "text",
      "text": "15"
    }
  ]
}
```

### evaluate_matlab_expression

Evaluate a MATLAB expression and return the result.

**Request:**
```json
{
  "name": "evaluate_matlab_expression",
  "arguments": {
    "expression": "sqrt(144)"
  }
}
```

**Response:**
```json
{
  "content": [
    {
      "type": "text",
      "text": "12"
    }
  ]
}
```

### check_matlab_code

Assess MATLAB code for style and correctness using Code Analyzer.

**Request:**
```json
{
  "name": "check_matlab_code",
  "arguments": {
    "code": "function y = myFunc(x)\ny = x * 2\nend"
  }
}
```

**Response:**
```json
{
  "content": [
    {
      "type": "text",
      "text": "Line 2: Add a semicolon after the statement to hide the output (when it is not the intent)."
    }
  ]
}
```

## Common Usage Patterns

### Basic Script Execution

```matlab
% Simple calculation
A = [1 2 3; 4 5 6; 7 8 9];
eigenvalues = eig(A);
disp(eigenvalues);
```

### Working with Files

```matlab
% Save data to file
data = rand(100, 3);
save('mydata.mat', 'data');

% Load and process
load('mydata.mat');
mean_values = mean(data);
writematrix(mean_values, 'results.csv');
```

### Plotting and Visualization

```matlab
% Create and save a plot
x = linspace(0, 2*pi, 100);
y = sin(x);
figure;
plot(x, y);
title('Sine Wave');
xlabel('x');
ylabel('sin(x)');
saveas(gcf, 'sine_plot.png');
```

### Matrix Operations

```matlab
% Linear algebra operations
A = magic(5);
b = sum(A, 2);
x = A \ b;  % Solve Ax = b

% Check solution
residual = norm(A*x - b);
fprintf('Residual: %.2e\n', residual);
```

### Signal Processing

```matlab
% Generate and filter signal
Fs = 1000;  % Sampling frequency
t = 0:1/Fs:1-1/Fs;
signal = sin(2*pi*50*t) + 0.5*randn(size(t));

% Apply low-pass filter
[b, a] = butter(6, 100/(Fs/2));
filtered = filter(b, a, signal);

% Compute FFT
Y = fft(filtered);
P2 = abs(Y/length(filtered));
P1 = P2(1:length(filtered)/2+1);
```

### Data Analysis

```matlab
% Statistical analysis
data = readtable('data.csv');
summary_stats = grpstats(data, 'Category', {'mean', 'std', 'median'});

% Correlation analysis
R = corrcoef(data{:, 2:end});

% Linear regression
mdl = fitlm(data, 'ResponseVar ~ Predictor1 + Predictor2');
disp(mdl);
```

### Custom Functions

```matlab
% Define reusable function
function [mean_val, std_val] = analyzeData(data)
    mean_val = mean(data, 'omitnan');
    std_val = std(data, 'omitnan');
    
    % Visualize
    figure;
    histogram(data, 30);
    title(sprintf('Mean: %.2f, Std: %.2f', mean_val, std_val));
end

% Use the function
results = rand(1000, 1) * 100;
[m, s] = analyzeData(results);
```

### Simulink Integration

```matlab
% Load and simulate Simulink model
load_system('mymodel');
simOut = sim('mymodel', 'StopTime', '10');

% Extract and plot results
time = simOut.tout;
output = simOut.yout;
plot(time, output);
```

## Troubleshooting

### MATLAB Not Found

**Problem:** Server cannot locate MATLAB installation.

**Solution:**
```bash
# Explicitly specify MATLAB root
--matlab-root=/Applications/MATLAB_R2026a.app  # macOS
--matlab-root=/usr/local/MATLAB/R2026a         # Linux
--matlab-root=C:\\Program Files\\MATLAB\\R2026a  # Windows

# Or set environment variable
export MW_MCP_SERVER_MATLAB_ROOT=/usr/local/MATLAB/R2026a
```

### Connection to Existing Session Fails

**Problem:** Cannot connect with `--matlab-session-mode=existing`

**Solution:**
1. Ensure MATLAB MCP Server Toolbox is installed:
```bash
./matlab-mcp-server --setup-matlab
```

2. In MATLAB, run:
```matlab
shareMATLABSession()
```

3. Verify connection status:
```matlab
status = shareMATLABSession('status')
```

### Path Issues

**Problem:** MATLAB cannot find scripts or data files.

**Solution:**
```matlab
% Check current directory
pwd

% Change directory
cd('/path/to/project')

% Add to path
addpath('/path/to/scripts');
addpath(genpath('/path/to/project'));  % Include subdirectories
```

### Graphics/Desktop Issues

**Problem:** Commands requiring GUI fail in `nodesktop` mode.

**Solution:** Graphics commands still work in `nodesktop` mode, but if issues persist:
```bash
# Switch to desktop mode
--matlab-display-mode=desktop
```

### Memory Issues

**Problem:** Out of memory errors with large datasets.

**Solution:**
```matlab
% Clear workspace
clear all

% Close figures
close all

% Use memory-efficient operations
% Instead of loading entire file:
data = load('largefile.mat');

% Use memory mapping:
m = memmapfile('largefile.dat', 'Format', 'double');
```

### Code Execution Timeout

**Problem:** Long-running code appears to hang.

**Solution:**
```matlab
% Add progress indicators
for i = 1:1000
    % Process
    if mod(i, 100) == 0
        fprintf('Progress: %d/1000\n', i);
    end
end

% Use parallel processing for large tasks
parfor i = 1:1000
    % Parallel computation
end
```

### Version Compatibility

**Problem:** Functions not available in older MATLAB versions.

**Solution:**
```matlab
% Check MATLAB version
ver('MATLAB')

% Conditional code based on version
if verLessThan('matlab', '9.10')  % R2021a
    warning('Some features require R2021a or later');
end
```

## Advanced Configuration

### Custom Tools Extension

Create custom MCP tools by providing a JSON extension file:

```bash
--extension-file=/path/to/my-tools.json
```

For details, see the [Custom Tools Guide](https://github.com/matlab/matlab-mcp-server/blob/main/guides/custom-tools.md).

### Multiple MATLAB Versions

```bash
# Development with latest MATLAB
claude mcp add --transport stdio matlab-dev -- /path/to/matlab-mcp-server --matlab-root=/usr/local/MATLAB/R2026a

# Production with stable MATLAB
claude mcp add --transport stdio matlab-prod -- /path/to/matlab-mcp-server --matlab-root=/usr/local/MATLAB/R2024b
```

### Project-Specific Configuration

For VS Code, use workspace-specific `.vscode/mcp.json`:

```json
{
    "servers": {
        "matlab": {
            "type": "stdio",
            "command": "/path/to/matlab-mcp-server",
            "args": [
                "--initial-working-folder=${workspaceFolder}",
                "--matlab-display-mode=nodesktop",
                "--initialize-matlab-on-startup=true"
            ]
        }
    }
}
```

## Resources

- [GitHub Repository](https://github.com/matlab/matlab-mcp-server)
- [MATLAB Agentic Toolkit](https://github.com/matlab/matlab-agentic-toolkit)
- [Simulink Agentic Toolkit](https://github.com/matlab/simulink-agentic-toolkit)
- [MathWorks AI Community](https://www.mathworks.com/matlabcentral/discussions/ai)
- [Model Context Protocol Specification](https://modelcontextprotocol.io)
