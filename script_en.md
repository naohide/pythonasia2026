# Talk Script - Fixit Linter + AI Coding

Total time: about 30 minutes (including Q&A)

## Time Plan

| Section | Slides | Time | Total |
|---|---|---|---|
| Opening & Agenda | 1-4 | 2:00 | 2:00 |
| Part 1 Intro + Background | 5-11 | 3:00 | 5:00 |
| What is Fixit | 12-16 | 2:30 | 7:30 |
| AST vs CST | 17-23 | 3:00 | 10:30 |
| AI Rule Generation | 24-28 | 2:30 | 13:00 |
| Practice | 29-37 | 4:00 | 17:00 |
| Part 2 Intro + AI Code Quality | 38-43 | 2:30 | 19:30 |
| Quality Guard with Fixit | 44-47 | 1:30 | 21:00 |
| mutmut | 48-56 | 3:00 | 24:00 |
| CI/CD Integration | 57-58 | 1:30 | 25:30 |
| Summary & Closing | 59-61 | 1:00 | 26:30 |

---

<!-- ============================================ -->
<!-- Opening & Agenda (2:00) Total 0:00-2:00 -->
<!-- ============================================ -->

## Slide 1: Title

Hello everyone. My talk title is "Fixit Linter + AI Coding — Python Code Quality Strategy for the AI Era."

Thank you for coming to PythonAsia 2026. In the next 30 minutes, I will show you a practical way to use AI and custom linters together. You can start using these ideas tomorrow. Please stay with me until the end.

---

## Slide 2: About Me

Let me introduce myself. My name is Hide. I work at Tokyo Gas as a Python developer. ~~You may think energy companies and Python are a strange match. But recently, we have been building most of our backend systems with Python.~~

Today, I will talk about a practical approach that came from real problems in our team. It combines AI and custom linters to keep code quality high.

I know this is sudden, but I'd like to apologize first. Because I'm not English native speaker, so please let me read my scripts, and if you can not understand my English, please read this slides later on the github.

Anyway, let's get started!

---

## Slide 3: Agenda

This talk has two parts.

In Part 1, I will explain why we need custom linters. Then I will introduce Fixit, a custom lint framework. I will also explain CST, the technology behind Fixit. After that, I will show how AI can generate lint rules. And I will walk you through a practical example of migrating from logging to structlog.

---

## Slide 4: Agenda (cont.)

In Part 2, I will change the focus. I will talk about code quality problems in AI-generated code. I will show how Fixit can guard against these problems. Then I will introduce mutmut, a mutation testing tool, to check test quality. Finally, I will cover CI/CD integration.

---

<!-- ============================================ -->
<!-- Part 1 Intro + Background (3:00) Total 2:00-5:00 -->
<!-- ============================================ -->

## Slide 5: Part 1 Start

Let's start Part 1. The main idea is: "Custom linters are more important than ever in the AI era."

In the last few years, AI coding tools became very popular. AI can generate a lot of code very fast. But how do we keep the quality high? That is the new challenge and custom linter is the one of the answers.

---

## Slide 6: 01. Background - Section Start

First, let me explain why we need custom linters.

Every team has rules. "Please write code this way." "Please don't use this library." The problem is: how do you make everyone follow these rules?

---

## Slide 7: Question for You

Here is a question for you. How does your team follow coding rules?

Do you just write them in a document? Do you check in code reviews? Or do you have some automation?

(Pause for 2-3 seconds)

I think many of you will say "code reviews." That is very natural. But code reviews have some problems.

---

## Slide 8: Code Review Today

Many teams try to follow coding rules through code reviews. But in reality, problems happen often. You can see them on the left side.

Rules become hidden knowledge. New members cannot learn them easily. Review quality depends on the reviewer. When people are busy, reviews become rough. The right side shows the ideal. Rules are clear and machines check them automatically.

How do we close this gap? That is today's topic. We should separate creative reviews for humans from routine checks for machines.

---

## Slide 9: A Common Scenario

This happens a lot in real teams. In a PR review, Reviewer A says "Please use structlog, not logging." But Reviewer B doesn't know this rule. Reviewer C says "Nobody told me this in my last PR."

The same comment repeats again and again. Every review has the same discussion. Reviewers feel stressed. Developers get confused because different reviewers say different things.

This is a clear sign. We should automate this. When humans cannot check the same way every time, let machines do it.

---

## Slide 10: Three Problems Custom Linters Solve

Custom linters solve three problems.

First: removing personal opinions. When you define rules as code, reviews become the same for everyone. Every reviewer gives the same feedback.

Second: enforcing best practices. Your team's best practices apply to the whole codebase automatically. Even new members get help from CI.

Third: moving from humans to machines. Machine checks are faster and more reliable. Human reviewers can focus on architecture and business logic.

---

## Slide 11: Limits of General Linters

You might think "Isn't ruff or flake8 enough?" These are great tools. I use ruff every day too.

But they only have general Python rules. They cannot check organization-specific requirements.

What kind of rules do teams need? Here are some examples. 

Using structlog instead of the logging module.   
Banning datetime.now() for better testing with clock injection.  
Using the company's HTTP client instead of requests, to keep authentication and retry settings the same. Like these.

These rules cannot be found by any general linter. ruff solves "general Python quality." Custom linters solve "your team's quality." That is why custom rules are needed.

---

<!-- ============================================ -->
<!-- What is Fixit (2:30) Total 5:00-7:30 -->
<!-- ============================================ -->

## Slide 12: 02. What is Fixit - Section Start

Now let me introduce Fixit. It is a framework for building custom linters easily.

---

## Slide 13: Fixit Overview

Fixit is a Python lint framework made by Instagram, now Meta. They built it to manage their large Python codebase. It is open source.

The biggest feature is that it uses LibCST, a CST library. I will explain CST later in detail. The key point is: it can change code without breaking comments or whitespace.

~~It is easy to write custom rules. It can not only find problems but also fix them automatically. You can set rules per project or per directory. This is useful for monorepos.~~

---

## Slide 14: Key Features & Basic Usage

Fixit has four main features. Lint finds problems. Fix repairs them automatically. This "find and fix" ability is very powerful. It doesn't just show problems. It also fixes them.

Local rules let you put rule files in your project. Config lets you set different rules for different directories. You don't need to install rules as a package. Just put them in your project folder.

Using Fixit is very simple. First, set the custom rules directory in `.fixit.toml`. You can also use `pyproject.toml`. And run these commands. That's all!

---

## Slide 15: Structure of a Fixit Rule

Let's look at the rule structure.

You make a class that extends `fixit.LintRule`. Then you write methods that start with `visit_`. This is the Visitor pattern. It walks through each node in the syntax tree and checks it. If you have used Python's `ast.NodeVisitor`, this will feel familiar.

You call `self.report` to report a problem. If you add a `replacement`, auto-fix works too. So one rule can do both "finding" and "fixing."

---

## Slide 16: Fixit Config File

The config file uses TOML format and like this. Please let me skip about detail.

---

<!-- ============================================ -->
<!-- AST vs CST (3:00) Total 7:30-10:30 -->
<!-- ============================================ -->

## Slide 17: 03. AST vs CST - Section Start

Next, I will explain CST, the core technology of Fixit. I will compare it with AST. This is the heart of Fixit's strength.

This part is a bit technical. But once you understand why CST matters, you will really see the value of Fixit.

---

## Slide 18: Two Ways to Read Source Code

There are two main ways to read source code with a program.

AST means Abstract Syntax Tree. CST means Concrete Syntax Tree.

The names look similar, but they keep very different amounts of information. This difference has a big effect on code transformation quality.

---

## Slide 19: AST vs CST Difference

Let's compare these. ~~AST is on the left. CST is on the right.~~

AST keeps only the structure. It throws away comments and whitespace. It keeps only "what the code does."

CST keeps everything. Structure, comments, whitespace, line breaks, indentation, the original format. It remembers "how the code was written."

If you only read code, there is no difference. But when you change code, the difference is huge.

---

## Slide 20: AST Basics

This is the AST diagram. Many of you have probably used Python's `ast` module.

AST takes only the "meaning" of code. It throws away the "look." This works well for checking code structure.

But for changing code, AST has a problem. It cannot bring back the original look.

---

## Slide 21: The AST Problem

Let me show you a real example.

The original code has comments like "# Initialize logger for this module." A team member wrote this comment for a reason.

When you change code with AST, the logging to structlog change works. But all comments disappear. Losing the team's notes is not OK in production.

---

## Slide 22: CST Basics

This is the CST diagram. CST keeps code in a form you can fully restore. If you parse code and turn it back, not a single whitespace or comment changes. That's why it is called "Concrete."

But lib2to3, which could handle CST, was removed from Python in version 3.13. So, we needed something new.

---

## Slide 23: LibCST is Born

Then LibCST came. Meta built this CST library to solve lib2to3's problems. It has the completeness of CST and the ease of AST.

It has three key features.  
First: the Visitor pattern for easy changes.  
Second: safe node changes with `with_changes()`.  
Third: clear rules for who owns whitespace and comments.

LibCST is the foundation of Fixit. Fixit can do safe auto-fixes because of LibCST.

---

## Slide 24: Clean git diff

Let's look at a real diff.

With CST-based changes, this is the same normal CST. Only the changed parts show in the diff. In this example, `import logging` changed to `import structlog`. The comments around it are fully kept. Review is very easy.

~~With AST-based changes, the worst case is the whole file gets rewritten. The diff becomes huge. Reviewers cannot tell what really changed. This is a big problem in production.~~

---

<!-- ============================================ -->
<!-- AI Rule Generation (2:30) Total 10:30-13:00 -->
<!-- ============================================ -->

## Slide 25: 04. AI Rule Generation - Section Start

Now it is AI's turn. I will show how to make custom linter rules from plain language.

You now know Fixit and LibCST are great. But you might think: "Isn't writing rules hard?" Let me answer that.

---

## Slide 26: Writing Fixit Rules is Not Easy

Fixit and LibCST are great tools. But honestly, writing rules needs understanding of LibCST's Visitor pattern and CST node structure. The learning cost is not small.

~~Most people can write Python. But most people don't know CST node types. That is completely normal.~~

---

## Slide 27: Old Barriers

Look at this LibCST node structure. You need to know many CST node types. You need to learn the difference between visit and leave methods. You need to learn with_changes for safe node changes. That's disgusting, isn't it.

---

## Slide 28: AI Rule Generation Flow

But with AI, this barrier drops a lot.

Step 1: Write what you want in plain words. "Replace logging with structlog."   
Step 2: AI makes the Fixit rule as Python code.  
Step 3: Run `fixit fix` to apply it to the whole codebase.

That is all. You don't need to know CST nodes. Just say what you want, and you get a rule.

---

## Slide 29: AI Lowers the Barrier

This is a big change. You don't need deep LibCST knowledge. ~~Just explain what you want in English or any language, and the rule is made.~~

What does this mean? Before, only a few members who knew CST could write rules. Now, everyone on the team can make and enforce coding rules. ~~If you think "I want to ban this pattern," you can make a rule right away.~~

---

<!-- ============================================ -->
<!-- Practice (4:00) Total 13:00-17:00 -->
<!-- ============================================ -->

## Slide 30: 05. Practice - Section Start

Now let me show you the practical part. I will automate the migration from logging to structlog.

~~Let me show the ideas I talked about with real code.~~

---

## Slide 31: Why structlog?

First, why structlog? Python's standard logging module is flexible. But structured log output is hard. You need to set up Handlers and Formatters for JSON output. Adding key-value pairs is also not easy.

With structlog, you can write key-value logs simply. It works well with JSON output and log tools like Datadog and Splunk. Many teams are moving to structured logging.

That's why I chose this example.

---

## Slide 32: What We Want to Change

Let's check what we want to change.

Before: `import logging` and `logging.getLogger(__name__)`. After: `import structlog` and `structlog.get_logger()`. We want to change both the import and the function call automatically.

~~It looks simple. But there are different styles like `from logging import getLogger` and `import logging as log`. Regular expressions cannot handle these well.~~

---

## Slide 33: AI Prompt Example

Here is an example prompt for AI. Write what you want clearly.

"Change `import logging` to `import structlog`. Change `logging.getLogger` to `structlog.get_logger`. Keep comments and formatting. ~~Add autofix so `fixit fix` can repair code automatically."~~

The tip is to write your needs as a list. If you write them in a vague way, AI will make vague rules.

---

## Slide 34: AI-Generated Fixit Rule (1/3)

These codes are made by AI. These are very complicated, so you don't need to understand all.

~~Let's look at the rule AI made. I will explain it in three parts.~~

~~The first part is the class. It extends `fixit.LintRule`. It has `MESSAGE` for the warning message, and `TAGS` for category tags. The structure is simple and clear.~~

~~AI made code that follows Fixit's rules correctly.~~

---

## Slide 35: AI-Generated Fixit Rule (2/3)

~~The second part finds and changes imports. The `visit_Import` method finds `import logging`.~~

~~The key point is `with_changes()`. This is LibCST's safe change API. It keeps the original comments and whitespace. It only changes the name part to `structlog`.~~

~~With AST, you would replace the whole node. Comments would disappear. With CST's `with_changes()`, only the needed change happens.~~

---

## Slide 36: AI-Generated Fixit Rule (3/3)

~~The third part finds and changes function calls. The `visit_Call` method finds `logging.getLogger(__name__)` and changes it to `structlog.get_logger()`.~~

~~The private method `_is_logging_get_logger()` keeps the check logic separate. Even if the check becomes complex, the visit method stays simple.~~

~~With these three parts, the rule can find and fix both import and function calls. The rule is complete.~~

---

## Slide 37: Testing the Rule

After writing a rule, we need tests. We cannot put AI code directly into production.

Fixit has a built-in test system. You list correct code in VALID and bad code in INVALID. That is how you write tests.

This is very simple. And just apply fixit fix with command. That's all. ~~You don't write test code. You just list sample code. This simplicity is one of Fixit's best points.~~

---

## Slide 38: Old Way vs Fixit + AI

Let's compare the old way with Fixit + AI. The difference is clear.

Old way: Manual find-and-replace misses things. Regular expressions cannot handle complex patterns. AST changes break comments. Writing CST rules needs expert knowledge.

Fixit + AI: CST-based safe changes keep all formatting. Pattern matching gives correct results. Plain language makes rules, so no expert knowledge needed. Applying to the whole codebase takes seconds.

Fixit + AI solves all the old problems.

---

<!-- ============================================ -->
<!-- Part 2 Intro + AI Code Quality (2:30) Total 17:00-19:30 -->
<!-- ============================================ -->

## Slide 39: Part 2 Start

Now let's start Part 2.

In Part 1, I talked about "using AI to make linter rules." In Part 2, I talk about "using linters to guard AI code." As AI use grows, this guard becomes more important.

---

## Slide 40: 06. AI Code Quality Problems - Section Start

Let me talk about new risks in the AI coding era.

GitHub Copilot, Claude Code, Cursor. How many of you use these tools? Probably many.

---

## Slide 41: How Much Do You Trust AI Code?

Here is another question. How much do you trust code written by AI?

(Pause for 2-3 seconds)

Do you think "if it works, it's fine"? Or do you "carefully review every time"? Honestly, many of us just accept AI output as it is. When code works, we don't always check quality. This is where the risk is.

---

## Slide 42: AI Coding - Good and Bad

AI coding has great benefits. It makes code very fast. It makes boilerplate automatically. It can use libraries you never used before. Development speed goes up for sure.

But look at the right side. AI does not know your team's rules. Context windows have limits. AI does not perfectly follow your team's coding guide. Tests may "just pass" but with low quality. There may be security problems.

Code is made faster. So the checking system must also be faster.

---

## Slide 43: Common Problems in AI Code

Here is a real example. This AI code works. Tests pass too.

~~But it should use structlog, not logging. It should use the company's HTTP client, not requests. Also, it should use clock injection, not datetime.now(), for better testing.~~

The code works, but it does not meet the team's rules. ~~AI follows general patterns from the internet. It does not know your team's best practices. This is the core of the "AI code quality problem."~~

---

## Slide 44: Why Custom Linters Matter in the AI Era

This is the heart of today's talk.

AI makes code. Linters correct code. These two are not enemies. They work together.

As AI writes more code, we need a system to automatically enforce team rules. Human reviewers cannot check everything.

Fixit can give the same feedback as a human reviewer. It works instantly. It never misses anything. The check runs the moment AI makes code. This is the "quality strategy for the AI era."

---

<!-- ============================================ -->
<!-- Quality Guard with Fixit (1:30) Total 19:30-21:00 -->
<!-- ============================================ -->

## Slide 45: 07. Quality Guard with Fixit - Section Start

Let's see how to use custom rules on AI code in practice.

---

## Slide 46: AI Coding + Fixit Workflow

The workflow has three steps.

First, AI makes code.  
Second, Fixit checks it right away with custom rules.  
Third, auto-fix the problems, or block the PR in CI.

When this cycle runs, AI code always meets team rules. Developers can let AI write code freely. Fixit keeps quality high automatically. It works like a "guardrail."

---

## Slide 47: Fixit on AI Code - Example

When you run `fixit lint`, you get results like this.

~~ReplaceLoggingWithStructlog: found logging usage. BanDatetimeNow: found datetime.now() usage. BanRequestsLibrary: found requests library usage.~~

~~Three problems found in one file. `fixit fix` repairs them all. AI output is automatically corrected to meet team rules. Developers don't need to remember every rule. The tool tells them.~~

---

<!-- ============================================ -->
<!-- mutmut (3:00) Total 21:00-24:00 -->
<!-- ============================================ -->

## Slide 48: 08. mutmut - Section Start

Next, let's talk about mutmut. The question is: "Are the tests for AI-made Fixit rules really good enough?"

---

## Slide 49: What is mutmut?

mutmut is a mutation testing tool for Python.

The idea is simple. It makes small changes to your code on purpose. Then it checks if your tests can find those changes. If found, the mutation is "Killed." If not found, it "Survived."

mutmut measures "the real quality of your tests." Code coverage alone cannot tell you this. Just running a line of code doesn't mean it works correctly.

---

## Slide 50: Are VALID / INVALID Really Enough?

I showed Fixit rule tests earlier. But are those VALID/INVALID cases really enough?

AI-made rules and test cases cover "common patterns." But they often miss edge cases. Mutation testing can show these gaps clearly.

---

## Slide 51: Review of Fixit Rule Tests

Let me review. VALID has `import structlog`. INVALID has `import logging`.

This looks enough at first. But think about it. Can it find `from logging import getLogger`? What about `import logging as log`? What about `import logging, os` with many modules?

We cannot tell from the tests alone if these cases are covered. This is where mutmut helps.

---

## Slide 52: Coverage vs Mutation Testing

Let me explain the difference between code coverage and mutation testing.

Code coverage only measures "which lines the test ran." Even without a single assert, running the code gives 100% coverage.

Mutation testing changes the rule code itself. Then it checks if VALID/INVALID can find the change. For example, if you change the string "logging," does the INVALID case still pass? If you flip a condition, does the VALID case get reported as a problem?

Mutation testing finds gaps in test cases directly. This is its big value.

---

## Slide 53: What mutmut Does to Fixit Rules

Let me show what mutmut does.

It changes the string "logging" to "XXloggingXX" in the rule. It makes conditions always True. It changes `and` to `or`. It automatically makes many small changes that break the rule logic.

If VALID/INVALID tests cannot find these changes, your test cases are not enough. If your rule logic breaks and tests don't notice, your tests are weak.

---

## Slide 54: How to Use mutmut

Using mutmut is also simple.

Run `mutmut run --paths-to-mutate=lint_rules/`. Check results with `mutmut results`. Survived: 3. This means 3 changes were not found by tests. Killed: 12. This means 12 changes were found correctly. Mutation score is 80%.

The 3 Survived are the weak points. ~~Use `mutmut show <id>` to see what change survived.~~ Then add test cases to cover those gaps.

---

## Slide 55: Adding Test Cases from Survived Mutants

Based on the Survived results, add test cases to VALID and INVALID like these.

~~Add to VALID: `from structlog import get_logger` (this is correct structlog usage). `import logging_utils` (has the word "logging" but is a different library).~~

~~Add to INVALID: `from logging import getLogger`. `import logging as log`. `import os, logging`.~~

This makes the rule much stronger. Adding cases that "have the word logging but are not problems" to VALID stops false alarms.

---

## Slide 56: Fixit Rule Strengthening Flow with mutmut

To summarize, the flow has three steps.

Step 1: AI makes the Fixit rule and VALID/INVALID test cases.  
Step 2: mutmut changes the rule code and checks if tests can find the changes.  
Step 3: Use Survived results to add more test cases.

Do this once or twice, and you get a strong custom linter. mutmut covers AI's weak points. This is a very smart combination.

---

<!-- ============================================ -->
<!-- CI/CD Integration (1:30) Total 24:00-25:30 -->
<!-- ============================================ -->

## Slide 57: 09. CI/CD Integration & Operations - Section Start

Finally, let me show how to put these tools into CI/CD for the whole team. Tools are only half useful if they are not part of your workflow.

---

## Slide 58: Integrating into the CI/CD Pipeline

Here is a GitHub Actions example.

For each PR, run `fixit lint .` for code quality check. Run `mutmut run` for rule test quality check. With just this simple setup, every PR is checked automatically.

~~If there are problems, CI turns red and the PR cannot be merged. Simple, but this is the best guardrail. Even new members who don't know the rules get help from CI.~~

---

<!-- ============================================ -->
<!-- Summary & Closing (1:00) Total 25:30-26:30 -->
<!-- ============================================ -->

## Slide 59: Summary

Here is the summary. I wanted to share four key points today.

1: AI can make custom linter rules from plain language. You can apply them to the whole codebase. No LibCST knowledge needed.

2: Fixit guards AI code quality. mutmut guards test quality. Together, they are the quality guardrail for the AI era.

3: Fixit + LibCST gives safe CST-based auto-changes. Comments and formatting are fully kept. git diff stays clean.

4: Add to CI/CD, and you can automatically guard code quality all the time, even in the AI coding era.

---

## Slide 60: Thank You!

Fixit + AI keeps your team's code quality safe automatically. If you are interested, please check the GitHub repository and the official Fixit documentation.

Thank you for listening.

(Wait for applause)
