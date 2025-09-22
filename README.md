# Workshop 2 — Legacy Migration Pt. 1
### Knowledge Base & Structure with Claude

> **DISCLAIMER**  
This “runs like magic,” but took ~3 days to craft :-).
Use it as a recipe.

---

## Fast Navigation

- [Setup](#setup)
- [Goal State](#goal-state)
- [Branch Map](#branch-map)
- [Artifacts](#artifacts)
    - [Audit Before Action](#audit-before-action)
    - [Generate Project Map](#generate-project-map)
    - [Document Unknown Patterns](#document-unknown-patterns)
    - [Test Coverage](#test-coverage)
    - [Create Migration Guide](#create-migration-guide)
    - [Document Database Schema](#document-database-schema)
    - [Inventory External Dependencies](#inventory-external-dependencies)
    - [Generate API Documentation](#generate-api-documentation)
- [Artifacts Checklist](#artifacts-checklist)

---

## Setup

### Repo Layout
```
/quickapps-cakephp3     # Legacy app (QuickApps CMS / CakePHP 3)
/quickapps-cakephp5     # Fresh CakePHP 5 app (clean target)
/migration-docs         # All generated docs live here
```

### Option 1: Run Apps from a ready docker-composer

```bash
git checkout master
```
```bash
cd quickapps-cakephp3
docker-compose up -d --build
cd ../quickapps-cakephp5
docker-compose up -d --build
```

> ⚠️ Port will not clash
**CakePHP3:**
- App `8080`
- Mysql: `3306`
**CakePHP5:**
- App `8081`
- Mysql: `3307`

### Option 2: Create both projects from scratch.
**Prompt: ON Plan Mode**
```
I'll start a project using quickAppsCMS (CakePHP 3.0) to be migrated to CakePHP 5.
Please check the repo https://github.com/quickapps/cms and install it locally (in a container).
Add all relevant steps from the installation into INSTALLATION.md.
Before generating any code, propose a full plan with milestones and risks.
```

**Prompt:**
```
Execute the plan
```

Now let's create cakephp5 fresh instance:

**Prompt:**
```
Perfect now create a quickapps-cakephp5 folder with a fresh cakephp5 project using docker with different ports from quickapps-cakephp3 project. We need both inside the project because we will migrate the code later.

```

**Prompt:**
```
/init
```

**Prompt:**
```
Do we need to keep the PROJECT_CONTEXT.md? If not remove it.
```


Then:
- Create `quickapps-cakephp5` fresh project with Docker
- Generate `PROJECT_CONTEXT.md` describing dual-app setup
- Decide if `PROJECT_CONTEXT.md` is needed long term

> **Note:** we have it ready to use on  `master` branch.

---

## Goal State
The result of this workshop will be available the `api_doc` branch:
Take a look before start
```bash
git checkout api_doc
```
Final artifacts live in `/migration-docs`.

---

## Branch Map

| Slide / Phase     | Branch name                                      | Artifacts                                                                 |
|-------------------|--------------------------------------------------|---------------------------------------------------------------------------|
| Setup / Context   | `master`                                         | `INSTALLATION.md`, (opt) `PROJECT_CONTEXT.md`                             |
| Audit             | `audit`                                          | `PROJECT_AUDIT.md`                                                        |
| Diagrams          | `diagrams`                                       | `PLUGINS_DEPENDENCY_MAP.md`, `CONTROLLERS_DEPENDENCY_MAP.md`, `CLAUDE.md` |
| Unknown Patterns  | `unknown_patterns` → `unknown_patterns_splitted` | `UNKNOWN_PATTERNS.md` → `unknown_patterns/*.md`                           |
| Test Plan         | `test_plan_10_cases` →    `test_plan`            | `TEST_PLAN.md` (+ subfiles)                                               |
| Migration Guide   | `migration_checklist`                            | `CakePHP_3_TO_5_GUIDE.md`                                                 |
| Database Schema   | `database_schema`                                | `DATABASE_SCHEMA.md`, `DATABASE_MIGRATION_ISSUES.md`                      |
| Dependencies      | `dependency_matrix`                              | `DEPENDENCY_MATRIX.md`                                                    |
| Business Rules    | `business_rules`                                 | `business-rules.md`                                                       |
| API Docs          | `api_doc`                                        | `API.md`, `swagger.yaml`                                                  |

---

## Artifacts

### Audit Before Action
```bash
git checkout master
```

**Prompt:**
```
The quickapps-cakephp3 will be migrated to Cakephp5. So I need to generate context files in /migration-docs folder. 
  Let's start with PROJECT_AUDIT.md. That file should contain a X-ray of the folder structure, files, plugins (and dependencies), config files, components, and handlers
At the beginning of PROJECT_AUDIT.md, list all sections and a summary of them to make it easier to understand the document.
```

> **Note:** check the result on: `audit` branch.
```bash
git checkout audit
```

---

### Generate Project Map
```bash
git checkout diagrams
```

**Create Visual Dependency Map**


**Prompt:**
```
Now lets create PLUGINS_DEPENDENCY_MAP.md. Create a Mermaid diagram showing the dependency relationships between QuickApps CMS plugins listed on project_audit.md
Show which plugins depend on others with arrows. Include whether dependencies are hard (required) or soft (optional).
```

**Controller Relationship Map**


**Prompt:**
```
Now lets create the CONTROLLERS_DEPENDENCY_MAP.  Analyze the controllers in src/Controller and each plugin's Controller folder. Create a Mermaid diagram showing:
  1. Controller inheritance hierarchy
  2. Shared traits or components
  3. Which controllers handle public vs admin routes
  4. API endpoints if any.
```


Document it on CLAUDE.md
**Prompt:**
```
Update CLAUDE.md with the files in migration-docs and their respective objectives.
```


> **Note:** check the result on: `diagrams` branch.
```bash
git checkout diagrams
```

### Document Unknown Patterns
```bash
git checkout diagrams
```

**Prompt:**
```
/clear
```

Here we can use 2 approaches:
1) Ask for Claude to identify unusual patterns and document them
2) Provide context, reference folders/files and get documentation from these sources.

On this example we'll follow the `Option 1`

**Prompt:**
```
Now we need to identify and document unknown patterns from quickapps-cakephp3 on the file UNKNOWN_PATTERNS.md, that are not common on cakephp3 framework, but its common on this project. Create adetailed explanation for developers unfamiliar with this pattern, using these questions as guide:
   1. What problem was it solving?
   2. How does it work?
   3. What would be the modern CakePHP 5 approach?
   4. What is the migration strategy from now to a modern solution

```

> Take a look on the result:
```bash
git checkout -b unknown_patterns
```


> ⚠️ UNKNOWN_PATTERNS.md is huge file, with more than 1500 lines. As best practice we should split it in smaller files.


**Prompt:**
```
Now we need to identify and document unknown patterns from quickapps-cakephp3 on the file UNKNOWN_PATTERNS.md, that are not common on cakephp3 framework, but its common on this project. Create adetailed explanation for developers unfamiliar with this pattern, using these questions as guide:
   1. What problem was it solving?
   2. How does it work?
   3. What would be the modern CakePHP 5 approach?
   4. What is the migration strategy from now to a modern solution

```
> **Note:** check the result on: `unknown_patter_splitted` branch.
```bash
git checkout unknown_patter_splitted
```
---

### Test Coverage
```bash
git checkout -b unknown_patter_splitted
```

**Prompt:**
```
 is there any test implemented on quickapps-cakephp3 ?
```

**Prompt:**
```
Perfect, please generate a TEST_PLAN.md, for unit, feature and e2e tests. So I can have 100% visibility of what should be working after the migration.
```


**Prompt:**
```
Lets split it in files per feature. Add the files, routes on backend, url on frontend and define a very detailed test plan 
```

> Take a look on the result, we have 10 test cases covering 400 scenários:
```bash
git checkout -b test_plan_10_cases
```

In this workshop we used only Claude code to define the test scenarios, so let's double-check the test approach with another prompt:

**Prompt:**
```
Please double check if there are controllers, models, or helpers not covered on the test plan.
```

> It's results on 15 test cases and 600 scenarios.
```bash
git checkout -b test_plan
```

---

### Create Migration Guide
```bash
git checkout -b test_plan
```

**Prompt:**
```
Please generate a CakePHP_3_TO_5_GUIDE.md. Listing all topics she should do make the code work on the new version of the framework. Components, functions, php methods, etc etc. WE should migrate to 3 to 5 directly of 3 to 4 then 5 ? 
Thinking hard on it and provide a perfect guide for even a junior developer to understand it.
```

> Check the result into `migration_checklist` branch:
```bash
git checkout -b migration_checklist
```

It looks perfect, but we need to add our knowledge on top of Claude Code. Give specifics to the model.
**Prompt:**
```
Is these areas below covered in CakePHP_3_TO_5_GUIDE?
  1. ORM changes (Table::find() differences)
  2. Request/Response object changes
  3. Middleware vs Dispatcher Filters
  4. Authentication/Authorization plugin changes
  5. FormHelper and HtmlHelper changes+
  6. Router changes
  7. Plugin loading mechanism
  8. Database driver changes

```

> Check the result into `migration_checklist_complete` branch:
```bash
git checkout -b migration_checklist_complete
```

 
---

### Document Database Schema
```bash
git checkout -b migration_checklist_complete
```

**Prompt:**
```
/clear
```

**Prompt:**
```
Analyze QuickApps-cakephp3 database schema and create comprehensive documentation on migration-docs/DATABASE_SCHEMA.md with the considerations to execute the migration to cakephp5 and mysql 8.
```

After generate, double-check the output:
**Prompt:**
```
Double-check the database schema versus the document to make sure all table and fields are documented.
```

You'll some migration details and detailed issues. Let's split it on two different files to work better on context files. Have one topic for /md is a good practice.
**Prompt:**
```
 Create another document called DATABASE_MIGRATION_ISSUES.md to detail all considerations and issues expected on the migration to quickapps-cakephp5 project provide more details on each consideration and update DATABASE_SCHEMA.md  to cover he structure only.
```
> Check the result into `database_schema` branch:
```bash
git checkout -b database_schema
```

---

### Inventory External Dependencies
```bash
git checkout -b database_schema
```

**Prompt:**
```
 A import part are the depencies to run in our cake5 project.
   Create a file /migration-docs/DEPENDENCY_MATRIX.md to Check CakePHP 5 compatibility for these QuickApps-cakephp3  dependencies from composer.json
   For each dependency:
   1. Is there a CakePHP 5 compatible version?
   2. What's the migration path?
   3. Are there modern alternatives?
   4. Can it be removed?
   Create a dependency compatibility matrix. 

```
> Check the result into `dependecy_matrix` branch:
```bash
git checkout -b dependecy_matrix
```
---


### Generate API Documentation
```bash
git checkout -b dependecy_matrix
```

**Prompt:**
```
Hey could you generate a API.md and a swagger file with filters request and response details for all routes in the quickapps-cakephp3 project o migration-docs folder?
```
> Final version of this workshop is on `api_doc` branch:
```bash
git checkout -b api_doc
```
---

## Artifacts Checklist

- [ ] `PROJECT_AUDIT.md`
- [ ] `PLUGINS_DEPENDENCY_MAP.md`
- [ ] `CONTROLLERS_DEPENDENCY_MAP.md`
- [ ] `CLAUDE.md`
- [ ] `UNKNOWN_PATTERNS.md` or `unknown_patterns/*.md`
- [ ] `TEST_PLAN.md` (+ subfiles on teste-plans/*.md)
- [ ] `CakePHP_3_TO_5_GUIDE.md`
- [ ] `DATABASE_SCHEMA.md`
- [ ] `DATABASE_MIGRATION_ISSUES.md`
- [ ] `DEPENDENCY_MATRIX.md`
- [ ] `API.md`, `swagger.yaml`

---
