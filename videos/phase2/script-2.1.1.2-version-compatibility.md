# Video Script: SDK Version Compatibility
## Module 2.1.1.2 - SDK Fundamentals

**Duration:** 20 minutes
**Target Audience:** Developers managing SDK versions
**Prerequisites:** Module 2.1.1.1 completed, SDK installed
**Learning Outcomes:** Effectively manage SDK versions and handle upgrades

---

## Scene 1: Introduction (1.5 minutes)

### Visual: Title slide

**[On camera - Instructor]**

Welcome back! In our last module, we successfully installed the UiPath Python SDK. Today, we're tackling a critical skill that many developers overlook until it causes problems: version management.

### Visual: Split screen - dev vs production environments showing version mismatch

**[Voice over visual]**

Picture this: Your code works perfectly in development. You deploy to production. The application crashes. After hours of debugging, you discover the root cause - a version mismatch between your development and production SDK versions.

### Visual: Learning objectives slide

**[On camera - Instructor]**

In this video, we'll master version management. You'll learn:
- How semantic versioning works and why it matters
- Multiple ways to check your SDK version
- How to manage different versions across projects
- Safe strategies for handling upgrades and breaking changes

Let's dive in!

---

## Scene 2: Understanding Semantic Versioning (3 minutes)

### Visual: Animated diagram of version numbers

**[On camera - Instructor]**

The UiPath SDK follows a standard called Semantic Versioning, or SemVer for short. Understanding SemVer is essential for managing any modern software dependency.

### Visual: Animated breakdown of version 0.2.1

**[Voice over animation]**

A semantic version has three parts separated by dots: MAJOR.MINOR.PATCH.

Let's break down version 0.2.1:

**The first number - 0 - is the MAJOR version.** This increments when there are breaking changes - changes that are incompatible with previous versions. Going from 0.9 to 1.0, or 1.5 to 2.0 are major version changes.

**The second number - 2 - is the MINOR version.** This increments when new features are added in a backward-compatible way. Your existing code keeps working, but new capabilities are available.

**The third number - 1 - is the PATCH version.** This increments for backward-compatible bug fixes. No new features, no breaking changes, just fixes.

### Visual: Traffic light diagram - green for patch, yellow for minor, red for major

**[On camera - Instructor]**

Think of it like a traffic light. Patch versions are green - safe to update without thinking. Minor versions are yellow - proceed with awareness, but generally safe. Major versions are red - stop and carefully review changes before updating.

### Visual: Special callout highlighting 0.x.y versions

**[Voice over callout]**

There's one critical exception: versions below 1.0.0 are considered unstable and experimental. In 0.x versions, even MINOR version changes might include breaking changes. The UiPath SDK is currently at version 0.x, so we need to be extra cautious with updates.

---

## Scene 3: Checking SDK Versions (2 minutes)

### Visual: Screen recording - terminal

**[On camera - Instructor]**

Let's look at three ways to check your SDK version. Each method is useful in different scenarios.

### Screen recording begins

**[Voice over screen recording]**

Method 1: The CLI check. This is the quickest:

```bash
uipath --version
```

You'll see output like "uipath, version 0.2.0". This confirms both the SDK installation and the version.

Method 2: Python code check. This is useful in scripts or when debugging:

```python
import uipath
print(f"SDK Version: {uipath.__version__}")
```

Notice the double underscore before and after "version" - that's the Python convention for version attributes.

Method 3: pip show command. This gives you the most detailed information:

```bash
pip show uipath
```

You see the version, dependencies, installation location, and more. This is invaluable when troubleshooting version conflicts.

---

## Scene 4: Version Constraints (4 minutes)

### Visual: requirements.txt file with different constraints

**[On camera - Instructor]**

Now let's talk about how to specify which SDK version your project needs. This goes in your requirements.txt file, and there are several ways to express version constraints.

### Visual: Side-by-side comparison of constraint types

**[Voice over visual]**

**Option 1: Exact version** - `uipath==0.2.0`

Two equals signs mean exactly this version. Use this when you need perfect reproducibility, like when debugging a specific issue. The downside? You miss out on bug fixes.

**Option 2: Minimum version** - `uipath>=0.2.0`

This means "0.2.0 or any newer version." Use this when you need specific features introduced in 0.2.0. The risk? You might automatically get breaking changes from major version updates.

**Option 3: Version range** - `uipath>=0.2.0,<1.0.0`

This is the sweet spot for most projects. You get new features and bug fixes automatically, but you're protected from breaking changes when version 1.0.0 is released.

**Option 4: Compatible release** - `uipath~=0.2.0`

The tilde-equals operator is shorthand for "compatible release." It's equivalent to `>=0.2.0,<0.3.0` - you get patch updates but not minor version updates.

### Visual: Recommendation highlight

**[On camera - Instructor]**

For production code, I recommend the version range approach: `uipath>=0.2.0,<1.0.0`. It balances safety with staying current.

### Screen recording - creating requirements.txt

**[Voice over screen recording]**

Let's create a requirements.txt:

```bash
echo "uipath>=0.2.0,<1.0.0" > requirements.txt
```

And install from it:

```bash
pip install -r requirements.txt
```

This ensures everyone on your team uses a compatible SDK version.

---

## Scene 5: Managing Multiple Versions (2.5 minutes)

### Visual: Diagram showing multiple projects with different SDK versions

**[On camera - Instructor]**

What if you're maintaining multiple projects that need different SDK versions? This is where virtual environments really shine.

### Screen recording - two terminal windows side by side

**[Voice over screen recording]**

Watch this. I have two projects.

In Project A, using SDK 0.1.9:

```bash
cd project-a
source venv/bin/activate
pip list | grep uipath
```

We see uipath 0.1.9.

Now in Project B, using SDK 0.2.0:

```bash
cd project-b
source venv/bin/activate
pip list | grep uipath
```

We see uipath 0.2.0.

### Visual: Diagram emphasizing isolation

**[On camera - Instructor]**

They coexist peacefully because each virtual environment is completely isolated. This is why we always use virtual environments - version conflicts simply don't exist when environments are isolated.

---

## Scene 6: Upgrading Safely (4 minutes)

### Visual: "Safe Upgrade Process" flowchart

**[On camera - Instructor]**

Upgrading your SDK is inevitable - you'll want bug fixes and new features. But upgrades need to be methodical to avoid breaking your application. Let me walk you through a safe upgrade process.

### Screen recording begins

**[Voice over screen recording]**

Step 1: Check your current version.

```bash
uipath --version
```

Let's say we're on 0.1.9.

Step 2: Check if updates are available.

```bash
pip list --outdated | grep uipath
```

This shows we can upgrade to 0.2.0.

### Visual: Browser showing GitHub releases page

**[Voice over browser]**

Step 3 - and this is critical - ALWAYS review the release notes before upgrading.

Navigate to the SDK's GitHub repository and check the releases page. Look for:
- What's new - new features you might want to use
- Bug fixes - problems that are now resolved
- Breaking changes - this is the most important section
- Deprecation notices - features being phased out

### Back to screen recording

**[Voice over]**

Step 4: Create a feature branch. Never upgrade directly on main.

```bash
git checkout -b upgrade-sdk-0.2.0
```

Step 5: Perform the upgrade.

```bash
pip install --upgrade uipath
```

Step 6: Verify the new version.

```bash
uipath --version
```

We now see 0.2.0.

Step 7: Run your test suite.

```bash
pytest
```

### Visual: Test results showing some failures

**[Voice over]**

If tests fail, this is your warning system. The failures tell you exactly what broke. Review the breaking changes in the release notes and update your code accordingly.

### Visual: Green passing tests

Once all tests pass, Step 8: Update your requirements.txt:

```bash
echo "uipath>=0.2.0,<1.0.0" > requirements.txt
pip freeze > requirements-lock.txt
```

Step 9: Commit and deploy through your normal process.

**[On camera - Instructor]**

This methodical approach might seem like overkill, but it prevents production outages. I've seen teams skip these steps and regret it.

---

## Scene 7: Handling Breaking Changes (2 minutes)

### Visual: Code comparison - old vs new API

**[On camera - Instructor]**

Let's look at a real example of handling breaking changes between versions.

### Visual: Side-by-side code comparison

**[Voice over visual]**

Imagine in version 0.1.x, starting a job looked like this:

```python
from uipath.orchestrator import start_job
result = start_job(release_key, robot_id)
```

But in version 0.2.x, the API changed:

```python
from uipath.orchestrator import start_job
result = start_job(
    release_key=release_key,
    strategy="Specific",
    robot_ids=[robot_id]  # Now a list!
)
```

### Visual: Migration checklist

**[Voice over]**

Notice the changes:
1. Parameters are now keyword arguments
2. There's a new required `strategy` parameter
3. `robot_id` became `robot_ids` and takes a list

**[On camera - Instructor]**

When you encounter breaking changes, search your codebase for all usage of the changed API, update each location, and test thoroughly. Most IDEs have find-and-replace features that make this easier.

---

## Scene 8: Lock Files for Reproducibility (2 minutes)

### Visual: Diagram showing dependency tree

**[On camera - Instructor]**

Here's a subtle problem many developers don't realize until it bites them. When you install the SDK, you're not just installing the SDK - you're installing all its dependencies too.

### Visual: Animated dependency tree growing

**[Voice over animation]**

The SDK depends on httpx, which depends on certifi, which depends on... you get the idea. If you just have `uipath>=0.2.0` in requirements.txt, the exact versions of these dependencies might vary between installations.

### Screen recording

**[Voice over screen recording]**

The solution is a lock file. Generate it like this:

```bash
pip freeze > requirements-lock.txt
```

### Visual: Comparing requirements.txt vs requirements-lock.txt

**[Voice over]**

requirements.txt might have 3-5 high-level dependencies. requirements-lock.txt has 30-50 dependencies with exact versions. This guarantees identical environments across development, testing, and production.

Install from the lock file:

```bash
pip install -r requirements-lock.txt
```

---

## Scene 9: Best Practices Recap (1.5 minutes)

### Visual: Best practices checklist appearing one by one

**[On camera - Instructor]**

Let's recap the version management best practices we've covered:

**[Voice over animated checklist]**

1. **Use version ranges in requirements.txt** - Balance safety and staying current
2. **Generate lock files** - Ensure reproducible builds
3. **Review release notes before upgrading** - Know what's changing
4. **Test in dev before deploying** - Catch issues early
5. **Use virtual environments** - Isolate project dependencies
6. **Document version requirements** - Help future maintainers
7. **Subscribe to release notifications** - Stay informed

---

## Scene 10: Hands-On Exercise Preview (30 seconds)

### Visual: Exercise instructions

**[On camera - Instructor]**

Time for your hands-on practice! Pause the video and complete this exercise:

1. Check your SDK version using all three methods
2. Create a requirements.txt with appropriate version constraints
3. Generate a lock file
4. Write a Python script that checks if the SDK version meets your requirements
5. If an update is available, practice the safe upgrade process

This should take about 20 minutes. The solution is in the course materials.

---

## Scene 11: Conclusion (1 minute)

### Visual: Key takeaways slide

**[On camera - Instructor]**

Excellent work! You now understand semantic versioning, know how to check and constrain SDK versions, and have a safe process for upgrades.

Version management might not be the most exciting topic, but it's the difference between a stable production system and 3 AM emergency debugging sessions.

### Visual: Next module preview

In our next module, we'll dive deep into dependency management with uv - learning how to achieve lightning-fast installs and resolve complex dependency conflicts.

### Visual: End card

Thank you for watching. Remember, version management is a skill that pays dividends across your entire career, not just with the UiPath SDK. See you in the next module!

---

## Production Notes

**Graphics needed:**
- Semantic versioning breakdown diagram
- Traffic light analogy for version types
- requirements.txt constraint comparison table
- Virtual environment isolation diagram
- Safe upgrade process flowchart
- Dependency tree visualization

**Screen recordings:**
- Version check methods (all three)
- Creating requirements.txt with different constraints
- Managing multiple projects with different versions
- Complete upgrade walkthrough
- Lock file generation and usage

**Code examples to display:**
- All version constraint formats
- Breaking change example (old vs new API)
- Version checking script

**Callouts/Annotations:**
- Highlight version numbers in output
- Point out breaking changes in release notes
- Annotate differences in API changes
- Mark steps in upgrade process

**Pacing notes:**
- Slow down during SemVer explanation - critical concept
- Allow time for viewers to understand version constraints
- Emphasize the upgrade process steps clearly
- Keep energy up during potentially dry lock file section

**Accessibility:**
- Clear captions for all technical terms
- High-contrast code displays
- Verbal description of all visual diagrams
