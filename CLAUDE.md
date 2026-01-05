# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a comprehensive fullstack developer interview preparation repository organized into topic-specific folders. It contains study materials, code examples, practice problems, and interview questions covering frontend (React.js, Next.js), backend (Node.js, Express.js, FastAPI), databases (PostgreSQL, MongoDB, DynamoDB), system design, cloud services (AWS), and coding challenges.

## Repository Structure

The repository is organized into numbered topic folders:

- **01-04**: Frontend topics (React, Next.js, Projects, Challenges)
- **05-08**: Backend topics (Node.js, Express, FastAPI, Projects)
- **09-12**: Database topics (PostgreSQL, MongoDB, DynamoDB, Design)
- **13-15**: System Design (Fundamentals, Problems, Examples)
- **16-19**: Coding Practice (LeetCode, Challenges, Data Structures, Algorithms)
- **20-23**: AWS & Cloud (Services, Serverless, DevOps, Architecture)
- **24-27**: Interview Preparation (Behavioral, Technical Questions, Mock Interviews, Portfolio)
- **28-30**: DevOps & Tools (Deployment, Testing, Setup)
- **31-32, 99**: Resources (Study Materials, Interview Experiences, Quick References)

Each folder contains:
- **README.md**: Comprehensive study guide with concepts, checklists, and resources
- **questions-and-answers.md**: Interview Q&A (where applicable)
- Code examples and implementations (embedded in markdown files)

## Content Format

All content is in Markdown format (`.md` files). The files follow consistent patterns:

1. **Study Guides (README.md)**: Structured with checklists, code examples, best practices, and progress trackers
2. **Q&A Files**: Interview questions with detailed answers, examples, and analogies
3. **Code Examples**: JavaScript/TypeScript, SQL, and configuration snippets embedded in markdown code blocks

## Working with This Repository

### When Adding New Content

1. Follow the existing folder numbering scheme (01-99)
2. Maintain consistent README.md structure:
   - Core concepts with checkboxes
   - Code examples in fenced code blocks
   - Common interview questions
   - Progress tracker at the end
3. Use clear section headers with emojis for visual organization
4. Include practical examples and real-world analogies in Q&A

### When Editing Existing Content

1. Preserve the checklist format (- [ ] items)
2. Maintain code block formatting with proper language identifiers
3. Keep the progress tracking sections intact
4. Ensure cross-references between related topics use correct folder names

### Study Roadmap Integration

The main README.md contains a phased study roadmap (8 weeks) and progress tracking. When adding new topics, consider how they fit into this learning progression.

### Git Workflow

- All commits should follow conventional commit message style (descriptive, focused on the "why")
- New content typically involves adding entire topic folders with README files
- The repository tracks interview preparation progress, so commit messages should reflect content completeness

## Key Technologies Covered

**Frontend**: React.js, Next.js, TypeScript, Redux, React Query
**Backend**: Node.js, Express.js, FastAPI, NestJS
**Databases**: PostgreSQL, MongoDB, DynamoDB, Redis
**Cloud**: AWS (EC2, Lambda, S3, RDS, DynamoDB, API Gateway)
**DevOps**: Docker, CI/CD, Terraform, CloudFormation
**Testing**: Jest, React Testing Library, Cypress

## Notes on Content Style

- Technical accuracy is prioritized over brevity
- Concepts include both theory and practical implementation
- Interview questions include complexity analysis and multiple approaches
- Code examples demonstrate best practices and common patterns
- Real-world analogies help explain complex concepts

## Documentation Enhancements

### Adding Table of Contents to Q&A Files

When working with `questions-and-answers.md` files, add a table of contents at the top with anchor links:

1. Create numbered list with question titles
2. Use markdown anchor format: `[Question Title](#q1-question-title-in-lowercase-with-hyphens)`
3. Place after the main heading, before first question
4. Separate TOC from content with `---`

Example format:
```markdown
# [Topic] Interview Questions & Answers

## Table of Contents

1. [Question 1](#q1-question-1)
2. [Question 2](#q2-question-2)

---

## Q1. Question 1
```

### Cross-Folder References

When referencing content across folders, use the numbered folder structure (e.g., "See 09-Database-PostgreSQL for indexing strategies").
- I need you to remmeber when i adding questions and answers you must update the table of contents in each file
- Before adding note analyze one or two similar files and foloow the same structure of the docs