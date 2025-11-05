# Video Script: Installing UiPath Python SDK
## Module 2.1.1.1 - SDK Fundamentals

**Duration:** 22 minutes
**Target Audience:** Developers new to UiPath Python SDK
**Prerequisites:** Basic Python knowledge, command-line familiarity
**Learning Outcomes:** Successfully install and verify UiPath SDK installation

---

## Scene 1: Introduction (2 minutes)

### Visual: Title slide with UiPath and Python logos

**[On camera - Instructor]**

Welcome to Module 2.1.1.1: Installing the UiPath Python SDK. I'm excited to help you take your first step into building automation and AI agents with UiPath.

In this video, we'll cover everything you need to get the UiPath Python SDK up and running on your system. Whether you're a seasoned Python developer or just starting your automation journey, by the end of this video, you'll have the SDK installed and verified.

### Visual: Learning objectives slide

**[Voice over slide]**

Here's what we'll accomplish together:
- Install the SDK using pip, Python's standard package manager
- Learn about uv, a modern high-speed alternative
- Verify our installation using multiple methods
- Troubleshoot common installation issues
- Apply industry best practices

Let's get started!

---

## Scene 2: Understanding the SDK (3 minutes)

### Visual: SDK architecture diagram

**[On camera - Instructor]**

Before we install anything, let's understand what the UiPath Python SDK actually is and why you'd want to use it.

The UiPath Python SDK is a comprehensive package that brings together everything you need to interact with UiPath services from Python code.

### Visual: Animated diagram showing SDK components

**[Voice over animation]**

When you install the uipath package, you get:

**First,** CLI tools - that's the uipath command you'll use from your terminal for tasks like initializing projects, running agents locally, packaging them, and publishing to Orchestrator.

**Second,** service clients - Python libraries for Orchestrator, AI Center, Document Understanding, and more. These let you programmatically interact with UiPath services.

**Third,** helper utilities - authentication helpers, type hints, async support, and deployment tools.

### Visual: Split screen - traditional automation vs Python SDK

**[On camera - Instructor]**

Think of it this way: UiPath Studio lets you build automation with a visual designer. The Python SDK lets you build automation and intelligent agents using Python code. Both are powerful - they're just different tools for different scenarios.

The SDK really shines when you're building:
- Intelligent agents with LangChain or LlamaIndex
- Custom integrations with UiPath services
- Automation that requires complex Python logic
- Data science workflows that need UiPath automation

---

## Scene 3: Python Requirements (2 minutes)

### Visual: Python version compatibility chart

**[On camera - Instructor]**

Before we install, let's make sure your system meets the requirements. The UiPath SDK requires Python 3.8 or higher, though I recommend 3.10 or later for the best experience.

### Visual: Terminal window

**[Screen recording - Terminal]**

Let's check your Python version. Open your terminal and type:

```bash
python --version
```

**[Voice over screen recording]**

You might need to use `python3` depending on your system:

```bash
python3 --version
```

If you see Python 3.8 or higher - great! You're ready to proceed. If not, head to python.org to download and install a recent version. I'll wait.

### Visual: Why Python 3.8+ slide

**[On camera - Instructor]**

Why Python 3.8 minimum? The SDK uses modern async/await features, improved type hints, and depends on packages that require Python 3.8+. Plus, older versions are no longer receiving security updates.

---

## Scene 4: Virtual Environments Explained (2 minutes)

### Visual: Diagram showing global vs virtual environment installation

**[On camera - Instructor]**

Now, before we install anything, we need to talk about virtual environments. This is crucial, so pay close attention.

### Visual: Animation showing package conflicts

**[Voice over animation]**

Imagine you have two projects: Project A needs UiPath SDK version 0.1.0, and Project B needs version 0.2.0. If you install both globally, you'll have a conflict. Only one version can be installed at a time.

Virtual environments solve this by creating isolated Python environments for each project. Each environment has its own packages, independent of others.

### Visual: Side-by-side comparison

**[On camera - Instructor]**

Think of virtual environments like separate houses. Each house has its own furniture (packages). Rearranging furniture in one house doesn't affect the others. That's isolation.

The Python community strongly recommends always using virtual environments for projects. We'll follow this best practice in all our work.

---

## Scene 5: Installation Method 1 - pip (5 minutes)

### Visual: Screen capture of terminal

**[On camera - Instructor]**

Let's do our first installation using pip - Python's standard package manager. I'll walk through every step.

### Screen recording begins

**[Voice over screen recording]**

First, let's create a project directory and navigate to it:

```bash
mkdir my-uipath-project
cd my-uipath-project
```

Now, create a virtual environment:

```bash
python -m venv venv
```

This creates a directory called 'venv' containing an isolated Python environment.

### Visual: File structure showing venv directory

**[Voice over]**

You'll see a new 'venv' folder with Python binaries and a lib directory for packages.

### Back to terminal recording

Now activate the virtual environment. On macOS or Linux:

```bash
source venv/bin/activate
```

On Windows:

```bash
venv\Scripts\activate
```

### Visual: Highlight command prompt change

**[Voice over]**

Notice your command prompt now shows (venv) - this tells you the virtual environment is active. All pip install commands now affect only this environment.

### Continue terminal recording

Now for the moment of truth - installing the SDK:

```bash
pip install uipath
```

### Visual: Installation progress

**[Voice over]**

Pip downloads the uipath package and all its dependencies. This typically takes 15-30 seconds depending on your internet connection.

### Visual: Completion message

You'll see "Successfully installed uipath-0.2.0" plus a list of dependencies.

Let's verify the installation:

```bash
uipath --version
```

### Visual: Version output

Perfect! We see the version number, confirming the CLI tools are installed and accessible.

---

## Scene 6: Installation Method 2 - uv (3 minutes)

### Visual: uv logo and performance stats

**[On camera - Instructor]**

Now let me show you a modern alternative: uv - a fast Python package installer written in Rust. It's 10 to 100 times faster than pip.

### Visual: Speed comparison graph

**[Voice over graphic]**

In benchmarks, operations that take pip 30 seconds complete in 2-3 seconds with uv. For large projects with many dependencies, this difference is substantial.

### Screen recording - terminal

**[Voice over screen recording]**

First, we need to install uv itself. On macOS or Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

On Windows PowerShell:

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Visual: Installation progress

This installs uv globally - it's the one tool I recommend installing globally because you'll use it across all projects.

### Continue screen recording

Now let's create a new project with uv:

```bash
mkdir uipath-uv-test
cd uipath-uv-test
uv venv
```

This creates a .venv directory - notice the dot prefix, a convention uv uses.

Activate it:

```bash
source .venv/bin/activate  # macOS/Linux
# or
.venv\Scripts\activate  # Windows
```

Now install the SDK:

```bash
uv pip install uipath
```

### Visual: Timer overlay showing speed

**[Voice over]**

Watch how fast this is. What took 30 seconds with pip completes in under 5 seconds.

Verify:

```bash
uipath --version
```

Perfect!

---

## Scene 7: Verification Methods (2 minutes)

### Visual: "Three Verification Methods" title

**[On camera - Instructor]**

We've already used one verification method - the CLI version check. Let me show you two more ways to confirm everything's working.

### Screen recording - Python script

**[Voice over]**

Method 2: Python import check. Create a file called verify_installation.py:

```python
import uipath

print(f"SDK Version: {uipath.__version__}")
print(f"Orchestrator client available: {hasattr(uipath, 'orchestrator')}")
print(f"AI Center client available: {hasattr(uipath, 'aicenter')}")
print("✓ SDK installed successfully!")
```

Run it:

```bash
python verify_installation.py
```

### Visual: Script output

We see the version and confirmation that key services are available.

### Back to terminal

**[Voice over]**

Method 3: pip show command:

```bash
pip show uipath
```

### Visual: pip show output

This displays detailed package information: version, dependencies, install location, and more. Very useful for troubleshooting.

---

## Scene 8: Common Issues & Solutions (2 minutes)

### Visual: "Troubleshooting" title card

**[On camera - Instructor]**

Let's quickly cover the most common installation issues and their solutions.

### Visual: Split screen - issue and solution

**[Voice over slides]**

**Issue 1: "Command not found"**

If `uipath --version` gives "command not found", your Python Scripts directory isn't in PATH.

Quick fix: Use `python -m uipath --version` instead.

Permanent fix: Add Python's Scripts directory to your system PATH.

**Issue 2: "Permission denied"**

This happens when trying to install globally without admin rights.

Solution: Always use virtual environments (which we're doing).

**Issue 3: "Python version too old"**

The error message will be clear: "requires Python >=3.8".

Solution: Upgrade Python from python.org or use pyenv for version management.

**Issue 4: "SSL Certificate errors"**

Common in corporate environments with proxy servers.

Solution: Configure your pip to use the corporate certificate authority, or use your company's internal PyPI mirror.

---

## Scene 9: Best Practices (1 minute)

### Visual: Best practices checklist

**[On camera - Instructor]**

Before we wrap up, let's review best practices you should always follow:

### Visual: Animated checklist

**[Voice over animation]**

1. **Always use virtual environments** - Never install SDK globally
2. **Pin versions in requirements.txt** - Ensures reproducibility
3. **Use lock files** - uv's lock files guarantee exact dependency versions
4. **Document your setup** - Future you will thank present you
5. **Keep Python updated** - But test before upgrading production code

---

## Scene 10: Hands-On Exercise Preview (30 seconds)

### Visual: Exercise instructions

**[On camera - Instructor]**

Now it's your turn! Pause the video and complete this hands-on exercise:

1. Create a new project directory
2. Set up a virtual environment
3. Install the UiPath SDK
4. Verify the installation with all three methods
5. Create a simple Python script that imports the SDK
6. Generate a requirements.txt file

This should take about 15 minutes. The exercise solution is in the course materials.

---

## Scene 11: Conclusion & Next Steps (30 seconds)

### Visual: Key takeaways slide

**[On camera - Instructor]**

Congratulations! You now have the UiPath Python SDK installed and verified. You've learned two installation methods, verified your setup, and know how to troubleshoot common issues.

### Visual: Next module preview

In our next module, 2.1.1.2, we'll explore SDK version compatibility - understanding semantic versioning, managing multiple SDK versions, and handling breaking changes.

### Visual: Course resources

Remember, all code examples, slides, and additional resources are available in the course repository.

### Visual: End card with social links

Thank you for watching, and I'll see you in the next module. Happy coding!

---

## Production Notes

**Graphics needed:**
- SDK architecture diagram
- Virtual environment illustration
- Speed comparison chart (pip vs uv)
- Python version compatibility chart
- Troubleshooting flowchart
- Best practices checklist

**Screen recordings:**
- Complete pip installation walkthrough
- Complete uv installation walkthrough
- All verification methods
- Troubleshooting examples

**Callouts/Annotations:**
- Highlight command prompt changes when venv activates
- Annotate installation times for pip vs uv comparison
- Point out version numbers in output

**Pacing notes:**
- Keep installation waiting time brief - speed up video during downloads
- Pause after each major step to let viewers follow along
- Use upbeat background music during installations to maintain energy

**Accessibility:**
- Ensure terminal text is large and clearly visible
- Use high-contrast color schemes
- Provide accurate closed captions for all technical terms
