# Resume Job Matcher

A privacy-focused **Django web application** that analyzes resumes, discovers relevant job opportunities from company career sources, evaluates candidate-job compatibility, identifies skill and experience gaps, and generates tailored ATS-friendly resumes.

## Overview

**Resume Job Matcher** is designed to simplify the early stages of the job application process.

The application takes a candidate's existing resume and turns it into a structured profile containing their skills, experience, education, certifications, projects, and relevant role information. It then searches supported company career sources and technology-park job listings for suitable opportunities.

For each relevant position, the system evaluates the candidate's profile against the job requirements and highlights areas that may need improvement. The candidate can then generate a tailored resume based on the selected opportunity.

The project follows a **privacy-first approach**, with uploaded resumes and derived data automatically removed after the configured retention period.

## Project Flow

```text
                    RESUME
                       │
                       ▼
              ┌─────────────────┐
              │ Resume Parsing   │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Profile Analysis │
              │                 │
              │ Skills          │
              │ Experience      │
              │ Education       │
              │ Certifications  │
              │ Projects        │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Job Discovery   │
              │                 │
              │ Career Sources  │
              │ ATS Feeds       │
              │ Tech Parks      │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Job Matching    │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
       Matching Areas       Skill / Experience
                            Gaps
              │                 │
              └────────┬────────┘
                       ▼
              ┌─────────────────┐
              │ Resume Tailoring│
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ ATS Resume      │
              │ Generation      │
              └────────┬────────┘
                       │
                       ▼
                FINAL OUTPUT
```

## Core Features

### Resume Processing

* Supports **PDF, DOCX, and TXT** resumes
* Extracts and structures resume content
* Detects sections such as:

  * Skills
  * Experience
  * Education
  * Certifications
  * Projects
  * Contact information
* Calculates professional experience from employment history

### Job Discovery

The application retrieves job information from supported career sources including:

* Greenhouse
* Lever
* Ashby
* Workable
* JSON-LD `JobPosting` data
* Official technology-park job listings

The project maintains its career-source registry through YAML configuration, making it possible to add or update companies without changing the core application logic.

### Job Matching

The matching engine evaluates a candidate against available positions using factors such as:

* Job title
* Role family
* Skills
* Experience requirements
* Job function
* Industry
* Candidate experience level

Role-family matching also allows related titles to be considered rather than relying only on exact job-title matches.

### Gap Analysis

For matched positions, the application identifies areas where the candidate may not fully meet the requirements.

The analysis can highlight:

* Missing skills
* Experience gaps
* Required years of experience
* Missing certifications
* Related requirements

This information is then used to guide resume tailoring and improvement.

### Tailored Resume Generation

The application can generate a job-specific resume while keeping the candidate's original information as the source of truth.

The generated resume can include relevant skills selected by the candidate and certification information based on the candidate's actual status.


The application provides an editor where the generated LaTeX can be modified and compiled into a PDF.

### Technology- and Business-Park Job Search

The search system includes official job listings from technology/business hubs.

The current project includes dedicated handling for:

* Infopark
* Technopark
* Cyberpark

Park listings are processed using multiple extraction strategies to handle different website structures, including structured data, embedded JSON, tables, cards, APIs, and optionally browser-rendered pages.

### Performance & Caching

The application is designed to reduce repeated network requests through:

* Parallel company fetching
* On-disk caching
* Background cache warming
* Stale-while-revalidate behavior
* Job-detail fetching only when required
* Configurable source timeouts

This allows previously processed career sources to be reused instead of being fetched from scratch for every search.

## Technology Stack

| Area              | Technologies                                 |
| ----------------- | -------------------------------------------- |
| Backend           | Python, Django                               |
| Frontend          | HTML, CSS, JavaScript, Django Templates      |
| Resume Parsing    | PyPDF, python-docx                           |
| Web Requests      | HTTPX                                        |
| Web Parsing       | BeautifulSoup                                |
| Configuration     | YAML                                         |
| Resume Generation | python-docx, LaTeX                           |
| PDF Generation    | PDFLaTeX                                     |
| Job Sources       | Greenhouse, Lever, Ashby, Workable, JSON-LD  |
| Deployment        | PythonAnywhere                               |
| Automation        | GitHub Actions                               |
| Database          | Django ORM / SQLite-compatible configuration |

## Architecture

```text
resume_job_matcher/
│
├── config/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── matcher/
│   ├── services/
│   │   ├── parser.py
│   │   ├── analysis.py
│   │   ├── jobs.py
│   │   ├── builder.py
│   │   ├── latex.py
│   │   ├── catalog.py
│   │   ├── hubs.py
│   │   ├── park_sources.py
│   │   └── snapshot.py
│   │
│   ├── templates/
│   ├── static/
│   ├── models.py
│   ├── views.py
│   ├── middleware.py
│   └── urls.py
│
├── companies.yaml
├── industries.yaml
├── job_functions.yaml
├── it_parks.yaml
├── requirements.txt
└── manage.py
```

## Configuration-Driven Career Registry

One of the key design decisions is keeping career sources outside the application logic.

Companies are maintained through `companies.yaml`, where each source can define information such as:

```yaml
- name: Example Company
  ats: greenhouse
  token: example-company
  regions: [europe]
  industries: [technology]
  job_functions: [engineering, data_ai, product]
```

This makes the job-source system easier to maintain and extend.

The project also includes separate registries for:

* Job functions
* Industries
* Technology parks
* Company metadata

## Privacy

Privacy is built into the application workflow.

Uploaded files are stored using randomized filenames and are associated with the user's browser session rather than being publicly accessible.

Resume data, parsed information, and generated files are automatically removed after the configured retention period.

The deployed application currently communicates a **60-minute retention period**, with an option for immediate deletion.

## Local Development

### 1. Clone the repository

```bash
git clone <repository-url>
cd resume_job_matcher
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Apply migrations

```bash
python manage.py migrate
```

### 4. Start the development server

```bash
python manage.py runserver
```

The application will then be available through the local Django development server.

### 5. Run tests

```bash
python manage.py test matcher
```

## Registry Validation

The company and job-source configuration can be validated with:

```bash
python manage.py validate_registry
```

## Deployment

The application is configured for deployment on **PythonAnywhere**.

The repository also includes deployment documentation and a GitHub Actions workflow for maintaining job-source data.

## Design Principles

The project was developed around several principles:

**Privacy First**
Resume information should be temporary and protected.

**Source Transparency**
Job opportunities should come from identifiable career sources rather than relying entirely on third-party aggregators.

**Configurable Architecture**
Companies, industries, job functions, and technology parks should be maintainable through configuration files.

**Practical Matching**
The system should consider related role families and requirements instead of relying solely on exact keyword matches.

**ATS Compatibility**
Generated resumes should remain simple, readable, machine-parsable, and suitable for modern applicant tracking systems.

## Project Highlights

* Built with Django and Python
* Custom resume parsing pipeline
* Rule-based job matching engine
* Multi-source career-page integration
* Configurable company and industry registry
* Technology-park job aggregation
* Background job searching with progress tracking
* Multi-level caching strategy
* Tailored DOCX resume generation
* ATS-friendly LaTeX and PDF generation
* Privacy-focused temporary file handling
* PythonAnywhere deployment
* GitHub Actions integration

## Live Application

The project is deployed as a working web application:

**Resume Job Matcher**
[Open Live Application](https://resumematcher07.pythonanywhere.com)

---

## Project Status

**Status:** Active / Deployed

Resume Job Matcher is a working end-to-end application covering resume processing, job discovery, job matching, gap analysis, and tailored resume generation.
