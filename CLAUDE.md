<?xml version="1.0" encoding="utf-8"?>
<ruleset>

  <description>
    Best practices for software development.
    Apply all rules to every file (**/*).
  </description>
  <globs>
    <glob>**/*</glob>
  </globs>
  <alwaysApply>true</alwaysApply>

  <!-- General -->
  <rule id="project-info" type="pre-work">
    <title>Review Project Information</title>
    <description>Always read assets/project_info.txt before making changes.</description>
  </rule>

  <!-- Code Quality -->
  <rule id="hardcoding" type="code-quality">
    <title>Avoid Hard-coding</title>
    <description>Use configs, environment variables, or dependency injection instead of hard-coded values.</description>
  </rule>
  <rule id="relative-paths" type="code-quality">
    <title>Use Relative Paths</title>
    <description>Do not use absolute OS paths. Stick to project-relative paths.</description>
  </rule>
  <rule id="style" type="code-quality">
    <title>Follow Existing Style</title>
    <description>Respect linter, formatter, and directory conventions. Do not introduce new styles in legacy code.</description>
  </rule>
  <rule id="best-practices" type="code-quality">
    <title>Adhere to Best Practices</title>
    <description>Apply clean code, SOLID principles, secure patterns, and maintainable structures. Use established libraries (OAuth2, REST, GraphQL).</description>
  </rule>
  <rule id="dry" type="code-quality">
    <title>DRY Principle</title>
    <description>Eliminate duplicate code by extracting shared logic into reusable utilities.</description>
  </rule>
  <rule id="names" type="code-quality">
    <title>Meaningful Identifiers</title>
    <description>Use snake_case. Names must clearly convey intent.</description>
  </rule>

  <!-- Documentation -->
  <rule id="docstrings" type="documentation">
    <title>Add Docstrings</title>
    <description>Include purpose, parameters, return values, and errors in all functions.</description>
  </rule>
  <rule id="arguments" type="documentation">
    <title>Explain Arguments</title>
    <description>Provide rationale for each parameter in docstrings or comments.</description>
  </rule>

  <!-- Logging -->
  <rule id="function-logging" type="logging">
    <title>Function Logging</title>
    <description>Add start/finish logs with timestamps and context to every function.</description>
  </rule>
  <rule id="change-logging" type="logging">
    <title>Change Logging</title>
    <description>Each change must: 1) insert inline log, 2) update CHANGELOG.md, 3) sync assets/project_info.txt.</description>
  </rule>
  <rule id="fullstack-logging" type="logging">
    <title>Fullstack Logging</title>
    <description>Ensure frontend and backend logs can be correlated for end-to-end debugging.</description>
  </rule>

  <!-- Design & Performance -->
  <rule id="single-responsibility" type="design">
    <title>Single Responsibility</title>
    <description>Each function should perform one task only.</description>
  </rule>
  <rule id="vectorize" type="performance">
    <title>Vectorize & Optimize</title>
    <description>Use NumPy/pandas vectorized operations instead of explicit loops.</description>
  </rule>
  <rule id="scalability" type="design">
    <title>Design for Scalability</title>
    <description>Consider complexity, concurrency, and resource usage.</description>
  </rule>

  <!-- Maintenance -->
  <rule id="sync-project-info" type="maintenance">
    <title>Sync Project Info</title>
    <description>Always update “File Structure” in project_info.txt after file additions, renames, or deletions.</description>
  </rule>

  <!-- Quality Assurance -->
  <rule id="browser-check" type="qa">
    <title>Verify Browser Behavior</title>
    <description>Run unit/UI tests or manual checks after frontend edits.</description>
  </rule>

  <!-- Library Usage -->
  <rule id="libraries" type="library-usage">
    <title>Prefer Established Libraries</title>
    <description>Use stable, community-vetted packages instead of reinventing functionality.</description>
  </rule>

  <!-- Workflow -->
  <rule id="root-cause" type="workflow">
    <title>Root Cause First</title>
    <description>Analyze logs and identify the root cause before fixing bugs. Do not fix symptoms.</description>
  </rule>
  <rule id="senior-dev" type="workflow">
    <title>Act Like a Senior Developer</title>
    <description>Think critically, refactor proactively, and anticipate inefficiencies with architectural foresight.</description>
  </rule>

</ruleset>
