# Video Script: Dependency Management with uv
## Module 2.1.1.3 - SDK Fundamentals

**Duration:** 30 minutes
**Target Audience:** Developers managing complex dependencies
**Prerequisites:** Modules 2.1.1.1-2.1.1.2, uv installed
**Learning Outcomes:** Master dependency management with uv for reproducible builds

---

## Scene 1: Introduction (2 minutes)

### Visual: Title slide with dependency tree visualization

**[On camera - Instructor]**

Welcome to Module 2.1.1.3: Dependency Management with uv. This might be the most important module in Phase 2, because dependency management is where many projects run into trouble.

### Visual: Split screen - working dev environment vs broken production

**[Voice over visual]**

You've probably experienced this: your code works perfectly on your machine, but when a teammate clones the repo or you deploy to production, it breaks. The cause? Dependency version mismatches.

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, we'll master professional dependency management using uv. You'll learn:
- Why uv is revolutionizing Python package management
- How to create lock files for reproducible builds
- Strategies for resolving dependency conflicts
- Optimizing complex dependency trees
- Industry best practices

This is advanced material, but by the end, you'll manage dependencies like a pro. Let's begin!

---

## Scene 2: The Dependency Problem (3 minutes)

### Visual: Simple requirements.txt file

**[On camera - Instructor]**

Let's start by understanding why dependency management is challenging. Look at this simple requirements file.

### Visual: Animated expansion showing hidden complexity

**[Voice over animation]**

```txt
# requirements.txt
uipath>=0.2.0
langchain>=0.1.0
```

Looks simple, right? Just two packages. But watch what happens when we install these.

### Visual: Dependency tree expanding dramatically

uipath doesn't stand alone - it depends on httpx, pydantic, click, and more. langchain also depends on pydantic and httpx, plus tenacity, dataclasses-json, and others. Each of THOSE has dependencies. What started as 2 packages becomes 50+ packages installed.

### Visual: Version conflict diagram

**[On camera - Instructor]**

Now here's where it gets tricky. Let's say today, when you install, httpx version 0.25.0 is the latest. Everything works great. Six months later, a new developer joins your team and runs `pip install -r requirements.txt`. Now httpx 0.27.0 is the latest. They get different versions of dozens of packages than you have.

### Visual: "Works on my machine" meme

**[Voice over]**

Different versions mean different behaviors, different bugs, different edge cases. This is the "works on my machine" problem that has plagued software development forever.

**[On camera - Instructor]**

The solution? Lock files. And the best tool for managing them? uv. Let me show you why.

---

## Scene 3: Why uv Changes Everything (3 minutes)

### Visual: Performance comparison chart

**[On camera - Instructor]**

uv is a game-changer for Python dependency management. It's written in Rust and is dramatically faster than pip.

### Visual: Side-by-side timer comparison

**[Voice over]**

In real-world tests with a typical project:
- pip takes 45 seconds for a fresh install
- uv takes 3 seconds

That's 15 times faster. For cached installs, uv is even more impressive - 0.5 seconds vs pip's 30 seconds.

### Visual: List of uv advantages

**[On camera - Instructor]**

But speed isn't the only advantage:

**Parallel downloads** - uv downloads multiple packages simultaneously while pip goes one at a time.

**Smart dependency resolution** - uv uses advanced algorithms to find compatible version combinations quickly.

**Global caching** - Downloaded and built packages are cached globally and reused across projects.

**Better error messages** - When conflicts occur, uv explains them clearly.

### Visual: Install comparison animation

**[Voice over]**

All of this while being a drop-in replacement for pip. Any pip command works with uv - just replace `pip` with `uv pip`.

---

## Scene 4: Lock Files Explained (4 minutes)

### Visual: Two-file system diagram

**[On camera - Instructor]**

The key to reproducible builds is the two-file system: requirements.in and requirements.txt.

### Screen recording begins

**[Voice over screen recording]**

Let me show you how this works. First, create a requirements.in file with your high-level dependencies:

```bash
cat > requirements.in << EOF
uipath>=0.2.0,<1.0.0
python-dotenv>=1.0.0
EOF
```

### Visual: Highlight version ranges

Notice we use version ranges - "greater than or equal to 0.2.0, but less than 1.0.0". We're expressing our requirements: we need at least 0.2.0, but we want to avoid breaking changes from version 1.0.

Now comes the magic - compile this into a lock file:

```bash
uv pip compile requirements.in -o requirements.txt
```

### Visual: Animated compilation process

Watch what uv does:
1. Reads your requirements
2. Fetches package metadata
3. Resolves all dependencies recursively
4. Finds compatible versions
5. Pins everything to exact versions

### Visual: Show generated requirements.txt

**[Voice over file display]**

Look at the result. requirements.txt now has 20+ packages with exact versions - including all the transitive dependencies we never specified directly.

```txt
uipath==0.2.0
httpx==0.25.0
certifi==2023.7.22
...and many more...
```

**[On camera - Instructor]**

This is your lock file. When anyone installs from this file, they get EXACTLY these versions. No surprises, no version drift, perfect reproducibility.

### Visual: Diagram showing lock file benefit

**[Voice over diagram]**

Developer A installs in January - gets these versions.
Developer B installs in June - gets the SAME versions.
Production deployment in December - SAME versions.

That's the power of lock files.

---

## Scene 5: The Complete Workflow (4 minutes)

### Visual: Workflow diagram

**[On camera - Instructor]**

Let me walk you through the complete workflow from project setup to deployment.

### Screen recording - full workflow demo

**[Voice over screen recording]**

Step 1: Create your project and virtual environment.

```bash
mkdir my-uipath-project
cd my-uipath-project
uv venv
source .venv/bin/activate  # macOS/Linux
```

Step 2: Create requirements.in with your dependencies.

```bash
cat > requirements.in << EOF
uipath>=0.2.0,<1.0.0
python-dotenv>=1.0.0
EOF
```

Step 3: Compile to generate the lock file.

```bash
uv pip compile requirements.in -o requirements.txt
```

### Visual: Highlight compilation output

Watch the output - uv is resolving dependencies incredibly fast. On my machine, this took 2 seconds. With pip, the equivalent operation takes 30+ seconds.

Step 4: Install from the lock file using sync.

```bash
uv pip sync requirements.txt
```

### Visual: Explain sync vs install

**[Voice over]**

I'm using `sync` instead of `install -r`. Here's why: sync ensures the environment EXACTLY matches the lock file. If there are packages installed that aren't in the lock file, sync removes them. This prevents pollution from previous experiments.

Step 5: Commit both files to version control.

```bash
git add requirements.in requirements.txt
git commit -m "Add dependency lock files"
```

### Visual: GitHub/Git visualization

**[On camera - Instructor]**

Commit both files. requirements.in shows your intentions - what you decided to depend on. requirements.txt shows the reality - exactly what gets installed.

---

## Scene 6: Updating Dependencies (3 minutes)

### Visual: Update workflow diagram

**[On camera - Instructor]**

Dependencies aren't static - packages get updates with bug fixes and new features. Let's see how to update safely.

### Screen recording

**[Voice over screen recording]**

To update all dependencies to their latest compatible versions:

```bash
uv pip compile --upgrade requirements.in -o requirements.txt
```

### Visual: Highlight --upgrade flag

The --upgrade flag tells uv to find the latest versions that satisfy your constraints. Without it, uv preserves existing versions when possible for stability.

After recompiling, review what changed:

```bash
git diff requirements.txt
```

### Visual: Diff showing version changes

You'll see version updates highlighted. Review these carefully - even compatible updates can introduce behavioral changes.

Test thoroughly:

```bash
uv pip sync requirements.txt
pytest
```

### Visual: Test results

**[Voice over]**

If your tests pass, you're good to commit. If they fail, you've caught an issue before it reaches production - that's the value of having tests!

**[On camera - Instructor]**

I recommend updating dependencies monthly in a dedicated branch, testing thoroughly, and only then merging to main.

---

## Scene 7: Handling Dependency Conflicts (4 minutes)

### Visual: Conflict scenario diagram

**[On camera - Instructor]**

Dependency conflicts are inevitable in complex projects. Let me show you how to handle them.

### Visual: Conflict example

**[Voice over visual]**

Here's a typical conflict:
- Package A requires httpx >= 0.25.0
- Package B requires httpx < 0.24.0

These requirements are incompatible - there's no version of httpx that satisfies both.

### Screen recording - triggering a conflict

**[Voice over screen recording]**

Let's create this conflict intentionally:

```bash
cat > requirements.in << EOF
package-a
package-b==1.0.0  # Old version with outdated httpx requirement
EOF

uv pip compile requirements.in -o requirements.txt
```

### Visual: Error message

**[Voice over error]**

uv detects the conflict and gives us a clear error:

```
error: Package 'httpx' has incompatible requirements:
  package-a requires httpx>=0.25.0
  package-b (1.0.0) requires httpx<0.24.0
```

**[On camera - Instructor]**

Now, how do we resolve this? Several strategies:

### Visual: Strategy flowchart

**[Voice over strategies]**

**Strategy 1: Update packages.** Try using --upgrade to get newer versions that might be compatible:

```bash
uv pip compile --upgrade requirements.in -o requirements.txt
```

Often, newer versions of packages update their dependencies.

**Strategy 2: Adjust your constraints.** Maybe package-b version 2.0 supports newer httpx:

```bash
cat > requirements.in << EOF
package-a
package-b>=2.0.0
EOF
```

**Strategy 3: Find alternatives.** If one package is particularly troublesome, consider replacing it with an alternative library.

**Strategy 4: Override (use cautiously).** As a last resort, you can force a specific version:

```bash
cat > requirements.in << EOF
package-a
package-b
httpx==0.24.1  # Forced compromise
EOF
```

### Visual: Warning sign

**[On camera - Instructor]**

Be very careful with overrides - you might create runtime issues. Test extensively.

---

## Scene 8: Development vs Production Dependencies (3 minutes)

### Visual: Environment separation diagram

**[On camera - Instructor]**

Professional projects separate development and production dependencies. Let me show you the pattern.

### Screen recording

**[Voice over screen recording]**

Create requirements.in for production - only what's needed to run:

```bash
cat > requirements.in << EOF
uipath>=0.2.0,<1.0.0
python-dotenv>=1.0.0
EOF
```

Create requirements-dev.in for development - includes testing and tooling:

```bash
cat > requirements-dev.in << EOF
-r requirements.in
pytest>=7.0.0
black>=23.0.0
mypy>=1.0.0
ruff>=0.1.0
EOF
```

### Visual: Highlight the -r inclusion

That `-r requirements.in` line includes all production dependencies, then adds development tools on top.

Compile both:

```bash
uv pip compile requirements.in -o requirements.txt
uv pip compile requirements-dev.in -o requirements-dev.txt
```

### Visual: Two lock files

**[Voice over]**

Now you have two lock files. In production, use:

```bash
uv pip sync requirements.txt
```

In development, use:

```bash
uv pip sync requirements-dev.txt
```

**[On camera - Instructor]**

This keeps production lean - no testing frameworks or linters bloating your deployment.

---

## Scene 9: uv Cache and Performance (2 minutes)

### Visual: Cache diagram

**[On camera - Instructor]**

Let me explain one of uv's secret weapons: global caching.

### Visual: Animated cache workflow

**[Voice over animation]**

When uv downloads and builds a package, it stores it in a global cache - typically at `~/.cache/uv` on macOS/Linux or `%LOCALAPPDATA%\uv\cache` on Windows.

When you install that same package in another project, uv doesn't download or build again - it copies from the cache. This is why cached installs are so incredibly fast.

### Screen recording - demonstrating cache benefit

**[Voice over screen recording]**

Watch this: I'll create two separate projects and install the SDK in both.

First project:

```bash
mkdir project1 && cd project1
uv venv && source .venv/bin/activate
time uv pip install uipath
```

4 seconds on my machine.

Second project:

```bash
cd ../
mkdir project2 && cd project2
uv venv && source .venv/bin/activate
time uv pip install uipath
```

0.3 seconds! That's 13x faster because everything was cached.

### Visual: Cache management commands

**[On camera - Instructor]**

Manage your cache with:
- `uv cache clean` - Free up disk space
- `uv cache dir` - Show cache location
- `uv cache info` - Show cache statistics

---

## Scene 10: Best Practices (2 minutes)

### Visual: Best practices checklist

**[On camera - Instructor]**

Let me share the best practices I've learned managing dependencies across dozens of projects.

### Visual: Practices appearing one by one

**[Voice over list]**

1. **Always use the two-file system** - requirements.in for intentions, requirements.txt for reality

2. **Commit both files to version control** - Never gitignore your lock file!

3. **Update regularly** - Monthly dependency updates prevent falling too far behind

4. **Test after updates** - Don't merge dependency updates without testing

5. **Separate dev and prod** - Keep production deployments lean

6. **Document the process** - Future maintainers will thank you

7. **Use sync not install** - Ensure exact environment reproduction

8. **Review what changed** - Always git diff requirements.txt after updating

**[On camera - Instructor]**

Follow these practices, and you'll avoid 90% of dependency issues.

---

## Scene 11: Hands-On Exercise Preview (1 minute)

### Visual: Exercise instructions

**[On camera - Instructor]**

Time for hands-on practice! Pause the video and complete this exercise:

1. Create a new project with requirements.in for uipath and python-dotenv
2. Generate a lock file with uv
3. Create requirements-dev.in with pytest, black, and mypy
4. Generate the development lock file
5. Practice updating all dependencies
6. Install pipdeptree and review your dependency tree
7. Practice resolving a simulated conflict

This should take about 25 minutes. The complete solution is in the course materials.

---

## Scene 12: Conclusion (1 minute)

### Visual: Key takeaways slide

**[On camera - Instructor]**

Excellent work! You've now mastered professional dependency management with uv. Let's recap what you learned:

**[Voice over takeaways]**

- uv provides 10-100x faster package operations
- Lock files ensure reproducible builds across all environments
- The two-file system separates intentions from implementation
- uv pip sync ensures perfect environment reproduction
- Regular updates and testing prevent dependency debt
- Conflicts are resolvable with systematic strategies

### Visual: Next module preview

**[On camera - Instructor]**

In our next module, 2.1.2.1, we shift gears to OAuth Authentication Flow. You'll learn how to authenticate with UiPath Orchestrator using External Applications, manage tokens, and handle refresh workflows.

### Visual: End card

This was a dense module, but dependency management is a foundational skill. You'll use these techniques in every Python project you work on.

Thank you for watching, and I'll see you in the next module!

---

## Production Notes

**Graphics needed:**
- Dependency tree visualization (expanding animation)
- uv vs pip performance comparison charts
- Two-file system diagram (requirements.in → requirements.txt)
- Lock file benefits diagram
- Workflow flowchart (complete process)
- Conflict resolution decision tree
- Cache architecture diagram
- Environment separation (dev vs prod)

**Screen recordings:**
- Complete lock file workflow from scratch
- Compilation process with timing
- sync vs install comparison
- Updating dependencies with --upgrade
- Simulating and resolving a conflict
- Dev vs prod separation setup
- Cache demonstration (two projects)
- pipdeptree visualization

**Code examples to display:**
- requirements.in samples
- Generated requirements.txt
- requirements-dev.in with -r inclusion
- Conflict scenarios
- uv commands with flags

**Callouts/Annotations:**
- Highlight compilation speed in real-time
- Point out version numbers in lock file
- Annotate differences between sync and install
- Mark conflict lines in error messages
- Show time saved with caching

**Pacing notes:**
- Slow down during lock file explanation - crucial concept
- Allow time to absorb the two-file system
- Keep energy up during conflict resolution
- Emphasize best practices clearly
- Demo should feel smooth and professional

**Accessibility:**
- Accurate captions for all technical terms
- Describe visual diagrams verbally
- High-contrast terminal display
- Large, readable text in code examples
