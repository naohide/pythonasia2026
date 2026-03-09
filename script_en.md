# Presentation Script - Fixit Linter + AI Coding

Estimated duration: ~30 minutes

---

## Slide 1: Title

Hello everyone. Today I'll be presenting "Fixit Linter + AI Coding — Python Code Quality Strategy for the AI Era."

---

## Slide 2: About Me

My name is Naohide Anahara. I work on Python development at Tokyo Gas. Today, I'd like to talk about a practical approach to maintaining code quality by combining AI with a custom linter.

---

## Slide 3: Agenda

Today's presentation is structured in two parts. In Part 1, I'll introduce Fixit, a custom lint framework, show how AI can automatically generate rules, and demo a migration from logging to structlog.

---

## Slide 4: Agenda (cont.)

In Part 2, we'll focus on quality issues in AI-generated code, how Fixit can protect against them, how mutmut verifies test quality, and how to integrate everything into CI/CD.

---

## Slide 5: Part 1 Section Start

Let's dive into Part 1. This is about why custom linters become even more important in the AI era.

---

## Slide 6: 01. Background & Challenges Section Start

First, let me explain the background — why custom linters are necessary.

---

## Slide 7: In Your Team... (Audience Question)

Here's a question for you all. How does your team enforce coding standards? Is it just written in a document? Pointed out in code reviews? Or do you have some kind of automation in place?

(Pause briefly)

---

## Slide 8: The Reality of Code Reviews

Many teams try to enforce standards through code reviews. But in reality, the problems on the left side are common — standards become tribal knowledge, review quality varies by reviewer. The right side is the ideal state, where standards are codified and mechanically verified. How to bridge this gap is today's theme.

---

## Slide 9: A Common Scenario

This is something I'm sure many of you have experienced. Reviewer A says "Use structlog instead of logging" on a PR. But Reviewer B doesn't know that rule. Reviewer C says "It wasn't flagged in the previous PR." The same feedback repeats over and over. This is exactly the kind of thing that should be automated.

---

## Slide 10: Three Problems Custom Linters Solve

Custom linters solve three problems. First, eliminating subjectivity — by defining standards as code, you prevent reviews from being person-dependent. Second, enforcing ideal patterns — your team's best practices are automatically applied across the entire codebase. Third, shifting from human to machine — checks that relied on humans are now done by machines, making them reliable and fast.

---

## Slide 11: Limitations of General-Purpose Linters

Now, you might be thinking "Aren't ruff or flake8 enough?" They're excellent tools, absolutely. But they only provide generic rules. They can't handle organization-specific requirements like "Use structlog instead of logging" or "Always add a specific decorator." That's why custom rules are essential.

---

## Slide 12: Examples of Organization-Specific Rules

What kind of rules are needed specifically? For example, standardizing from logging to structlog, banning direct use of datetime.now(), using an internal HTTP client instead of requests — these rules can never be detected by general-purpose linters.

---

## Slide 13: 02. What is Fixit Section Start

Now let me introduce Fixit, the tool that solves these challenges.

---

## Slide 14: Fixit Overview

Fixit is a Python lint framework developed by Instagram, which is part of Meta. Its key feature is being built on libcst, a CST library. It makes creating custom rules straightforward, comes with auto-fix capability built in — not just detection — and supports hierarchical configuration at the project and directory level.

---

## Slide 15: Key Features of Fixit

Fixit has four key features. Lint detects violations, Fix auto-corrects them. Local rules can be placed directly in your repository, and Config enables hierarchical settings. The auto-fix capability is particularly powerful — being able to detect and fix in a single command is a major advantage.

---

## Slide 16: Basic Usage of Fixit

Usage is very straightforward. You specify custom rule directories in `.fixit.toml`, run `fixit lint` to detect, and `fixit fix` to auto-correct. Adding the `--interactive` option lets you review and apply fixes one by one.

---

## Slide 17: Basic Structure of a Fixit Rule

Let's look at the rule structure. You inherit from `fixit.LintRule` and define methods starting with `visit_`. This is the Visitor pattern — it traverses each node of the syntax tree for inspection. `self.report` reports violations, and specifying `replacement` enables auto-fix.

---

## Slide 18: Fixit Configuration File

The configuration file is in TOML format. You can enable or disable rules and specify the Python version. Furthermore, placing a separate `.fixit.toml` in subdirectories lets you apply different rule sets per directory. This is extremely useful for monorepos.

---

## Slide 19: 03. AST vs CST Section Start

Next, I'll explain CST, the core of Fixit, and how it differs from AST. This is the technical foundation of Fixit's strength.

---

## Slide 20: Two Approaches to "Understanding" Source Code

There are two main approaches to parsing source code. AST — Abstract Syntax Tree, and CST — Concrete Syntax Tree. This difference has a significant impact on the quality of code transformations.

---

## Slide 21: AST vs CST Differences

On the left is AST, on the right is CST. AST retains only structural information, discarding comments and whitespace. CST, on the other hand, preserves everything — structural information plus comments, whitespace, newlines, and the original formatting.

---

## Slide 22: AST Basics

Here's the AST diagram. AST's design philosophy is to extract only the "meaning" of code while discarding the "appearance." It's convenient for code analysis, but problematic for code transformation.

---

## Slide 23: The Problem with AST

Let's look at a concrete example. The original code has comments. When transformed with AST, the conversion from logging to structlog works, but all comments are lost. Losing the team's notes and intentions is unacceptable in a production environment.

---

## Slide 24: CST Basics

Here's the CST diagram. CST represents code in a "fully recoverable form." However, the legacy lib2to3 was removed in Python 3.13, so a successor was needed.

---

## Slide 25: Advantages of CST

When performing the same transformation with CST, you can see that all comments and whitespace are preserved, with only the logging parts changed to structlog. This is the "safe transformation" required in practice.

---

## Slide 26: Challenges of lib2to3-Based CST

However, lib2to3-based CST had its own issues. Ownership of whitespace and comments was ambiguous, deleting nodes could generate invalid code, and complex transformations were cumbersome.

---

## Slide 27: The Birth of LibCST

That's where LibCST comes in. Developed by Meta, this CST library achieves both the completeness of CST and the usability of AST. It features a Visitor pattern-based transformation framework and safe node manipulation via `with_changes()`. And LibCST is the foundation of Fixit.

---

## Slide 28: LibCST Diagram

(Showing the diagram) Here's the architecture of LibCST.

---

## Slide 29: Why LibCST Matters in Production

When performing automated refactoring on a large codebase, losing comments or formatting is absolutely unacceptable. With LibCST, you can transform code reliably while maintaining the existing code style, and diffs remain clean for review.

---

## Slide 30: Clean git diffs

Let's look at an actual diff. With CST-based transformation, only the changed parts appear in the diff. Comments are preserved, making review much easier. With AST-based approaches, entire files can be rewritten, resulting in massive diffs.

---

## Slide 31: 04. AI-Powered Rule Generation Section Start

Now it's time for AI to enter the picture. I'll show you how to generate custom linter rules from natural language.

---

## Slide 32: Writing Fixit Rules Has a Surprisingly High Barrier

Fixit and LibCST are wonderful tools, but writing rules requires understanding libcst's Visitor pattern and CST node structure, which involves a significant learning curve.

---

## Slide 33: Traditional Barriers to Rule Creation

Take a look at this libcst node structure. You need to know the types and hierarchy of CST nodes, understand the difference between visit and leave methods in the Visitor pattern, and learn how to write safe node modifications with with_changes. Honestly, it's not something you can write quickly.

---

## Slide 34: AI-Powered Rule Generation Flow

But with AI, this barrier is dramatically lowered. Step 1: Describe in natural language — "Replace logging usage with structlog." Step 2: AI generates the Fixit rule. Step 3: Apply across the entire codebase with fixit fix. That's all there is to it.

---

## Slide 35: How AI Lowers the Barrier

This is a major paradigm shift. Without deep knowledge of libcst, you can generate rules just by explaining what you want in plain English or any language. Everyone on the team can now participate in defining and enforcing coding standards.

---

## Slide 36: Tips for Providing Context to AI

There are also tips for improving AI accuracy. Include in your prompt the basic LintRule structure, key libcst node types, Before/After examples, edge cases, and existing Fixit rule implementations. AI excels at pattern matching, so the better the examples you provide, the better the results.

---

## Slide 37: 05. Practical Demo Section Start

Let's move into the practical demo. We'll automate the migration from logging to structlog.

---

## Slide 38: Why structlog?

First, why structlog? Python's standard logging is flexible but makes structured log output cumbersome. With structlog, you can write concisely in key-value format, with great compatibility with JSON output and log pipelines. Many organizations are trending toward this migration.

---

## Slide 39: Transformation Goal

Let's confirm the transformation goal. The Before uses standard logging, the After uses structlog. We want to automatically handle both the import statement change and the function call change.

---

## Slide 40: Example Prompt for AI

The prompt to AI looks like this. Write specifically what you want to do: convert import logging to import structlog, convert logging.getLogger to structlog.get_logger, preserve comments and formatting, and implement autofix.

---

## Slide 41: AI-Generated Fixit Rule (1/3)

Here's the first part of the AI-generated rule — the class definition. It's a simple structure inheriting from `fixit.LintRule`, with a message and tags defined.

---

## Slide 42: AI-Generated Fixit Rule (2/3)

The second part is the import detection. The `visit_Import` method finds `import logging` and uses `with_changes()` to replace it with structlog. The key point is that with_changes preserves comments and whitespace while changing only the necessary parts.

---

## Slide 43: AI-Generated Fixit Rule (3/3)

The third part is function call detection. `visit_Call` finds `logging.getLogger(__name__)` and converts it to `structlog.get_logger()`. The detection logic is nicely separated into a private method — good structure.

---

## Slide 44: Testing the Rule

Once you've written a rule, you need tests too. Fixit has a built-in test framework — you simply list correct code in VALID and violating code in INVALID. Very intuitive.

---

## Slide 45: Bulk Application with Fixit Commands

Let's apply it. `fixit lint` finds violations in 23 files, and `fixit fix` corrects them all at once. Migrating 23 files completes in just seconds. I can't imagine how long it would take manually, but with this, it's instant.

---

## Slide 46: Traditional Approach vs Fixit + AI

Comparing the traditional approach with Fixit + AI, the difference is stark. Manual search and replace, regex workarounds, risk of comment destruction, need for specialized knowledge — all replaced by safe CST-based transformation, accurate detection, complete formatting preservation, and generation from natural language.

---

## Slide 47: Part 1 Summary: Impact

Here's the Part 1 summary. With AI, rule creation takes just minutes. Fixit fix corrects the entire codebase in seconds. And since it's CST-based transformation, formatting and comment preservation is 100%.

---

## Slide 48: Part 2 Section Start

Now we enter Part 2. We tackle a new challenge — how to protect the quality of code written by AI.

---

## Slide 49: 06. Quality Challenges in AI-Generated Code Section Start

Let me talk about the new risks in the AI coding era.

---

## Slide 50: AI-Written Code... (Audience Question)

Here's another question. How much do you trust the code AI writes?

(Pause briefly)

---

## Slide 51: The Light and Shadow of AI Coding

AI Coding does bring tremendous benefits. Dramatically faster code generation, automatic boilerplate creation, instant use of unfamiliar libraries. But look at the right side. AI doesn't know your organization's standards. It freely uses deprecated patterns, sometimes produces tests that "just pass," and may include security issues.

---

## Slide 52: Problematic Code AI Tends to Generate

Here's a concrete example. This AI-generated code works. But logging should be structlog, requests should be the internal client, datetime.now() has testability issues. It works, but it doesn't meet the team's standards.

---

## Slide 53: Why Custom Linters Matter in the AI Era

This is the core message of today's presentation. AI generates code. Linters correct code. As the volume of AI-written code increases, an automated system for enforcing organization-specific quality standards becomes essential. With Fixit, you can make the same checks a human reviewer would — instantly and without any gaps.

---

## Slide 54: 07. Guarding AI Code Quality with Fixit Section Start

Now let's see specifically how to apply custom rules to AI-generated code.

---

## Slide 55: AI Coding + Fixit Workflow

The workflow is three steps. First, AI generates code. Then Fixit immediately checks it with custom rules. Finally, it either auto-fixes or blocks the PR in CI. As this cycle runs, AI-generated code consistently meets team standards.

---

## Slide 56: Applying Fixit to AI-Generated Code

When you actually run fixit lint, results like these come up. ReplaceLoggingWithStructlog, BanDatetimeNow, BanRequestsLibrary. Three violations detected in one file, all fixed at once with fixit fix. AI output automatically corrected to meet team standards.

---

## Slide 57: Fixit Rule Examples for AI Code

Here are three useful rules to have for AI-generated code. Detection of deprecated libraries, enforcement of security patterns, and correction of coding standards. Security rules are especially important — they can prevent eval() calls and SQL string concatenation that AI tends to generate.

---

## Slide 58: 08. mutmut Section Start

Next, let's talk about mutmut — how to strengthen test cases for Fixit rules.

---

## Slide 59: What is mutmut?

mutmut is a mutation testing tool for Python. It intentionally introduces small changes to your code and checks whether your tests can detect them. It measures the "true quality" of your tests — something code coverage alone can't reveal.

---

## Slide 60: Are VALID / INVALID Really Enough? (Audience Question)

We saw Fixit rule tests earlier, but are those VALID/INVALID cases really sufficient? Mutation testing reveals gaps in test cases.

---

## Slide 61: Recap of Fixit Rule Tests

Let's recap. VALID has `import structlog`, INVALID has `import logging`. At first glance it seems sufficient, but what about `from logging import getLogger`? What about `import logging as log`? What about `import logging, os`? We can't tell if these cases are covered.

---

## Slide 62: Coverage vs Mutation Testing

Code coverage only measures "which lines were executed by tests" — it can reach 100% just by running lines. Mutation testing, on the other hand, mutates the rule's code itself and checks whether VALID/INVALID cases detect the mutation. It directly discovers gaps in test cases — that's the key difference.

---

## Slide 63: What mutmut Does to Fixit Rules

Specifically, mutmut changes the string "logging" to "XXloggingXX", makes conditions always True, swaps and with or, and so on. If VALID/INVALID can't detect these mutations, it means your test cases are insufficient.

---

## Slide 64: How to Use mutmut

Usage is simple. Run `mutmut run`, check results with `mutmut results`. Survived: 3 — mutations the tests couldn't detect. Killed: 12 — mutations correctly detected. Mutation score: 80%. The Survived cases are where you need to improve your test cases.

---

## Slide 65: Adding Test Cases from Survived Mutants

Based on the Survived results, add test cases to VALID and INVALID. Add `from structlog import get_logger` and `import logging_utils` to VALID, add `from logging import getLogger`, `import logging as log`, and `import os, logging` to INVALID. This dramatically improves rule robustness.

---

## Slide 66: Fixit Rule Strengthening Flow with mutmut

To summarize, the flow is three steps. AI generates Fixit rules and VALID/INVALID cases, mutmut performs mutation testing on the rule code, and Survived mutants inform additional test cases. Running this cycle produces a robust custom linter.

---

## Slide 67: Fixit + mutmut Combined

Fixit enforces code quality with custom rules, and mutmut verifies the test quality of those rules themselves. AI-generated rules and test cases may "work but aren't sufficient." By using mutmut to automatically detect gaps in test cases, you achieve a highly reliable custom linter.

---

## Slide 68: 09. CI/CD Integration & Operations Section Start

Finally, let's look at how to integrate all of this into CI/CD for team-wide operations.

---

## Slide 69: Integrating into the CI/CD Pipeline

Here's a GitHub Actions configuration example. For each PR, fixit lint checks code quality and mutmut run checks test quality. With just this configuration, code quality and test quality are automatically verified on every PR.

---

## Slide 70: Steps to Introduce to Your Team

Introduction is five steps. Define rules, generate with AI, verify test quality with mutmut, bulk-apply to existing code, and integrate with CI. Each step is independent, so I recommend starting with one rule and gradually expanding.

---

## Slide 71: Example Project Structure

The project structure looks like this. Custom rules go in the lint_rules directory, configuration in .fixit.toml, and mutmut settings in setup.cfg. It integrates naturally into your existing project structure.

---

## Slide 72: Example Implementation Timeline

Here's an actual implementation timeline. Day 1: organize your standards. Days 2-3: generate rules and tests with AI. Day 4: bulk-apply to existing code and run mutmut. Day 5: CI/CD integration. You can complete the entire introduction in one week.

---

## Slide 73: FAQ (1)

A frequently asked question: "Can AI-generated rules be used as-is?" Generally they work out of the box, but edge case testing is necessary. Use Fixit's built-in test framework to verify VALID/INVALID patterns before applying to production.

---

## Slide 74: FAQ (2)

Another one: "Isn't mutmut's execution time too long?" It supports parallel and incremental execution, so after the first run, only diffs need to be verified. In CI, targeting only changed files is practical.

---

## Slide 75: Summary

Let me summarize with four key points. Part 1: Generate custom linter rules from natural language with AI, and apply them across the entire codebase. Part 2: Guard AI-written code quality with Fixit, and test quality with mutmut. Safe automatic transformation with Fixit + LibCST preserves formatting and comments completely. And by integrating with CI/CD, you can continuously and automatically protect code quality even in the AI Coding era.

---

## Slide 76: Next Step

Finally, three steps to get started today. Install with pip install fixit mutmut. Choose one of your team's standards and ask AI to generate a rule. Apply it to existing code with fixit fix, and verify test quality with mutmut run. Please try it out starting today.

---

## Slide 77: Thank You!

Thank you for listening. Fixit + AI — automatically protecting your team's code quality. If you're interested, please check out the GitHub repository. I'm happy to take any questions.

(Wait for applause)
