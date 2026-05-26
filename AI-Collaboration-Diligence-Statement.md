# AI Collaboration Diligence Statement

> *A reflection on the responsible use of artificial intelligence in the development of this workshop and its documentation*

-----

## Overview

This workshop — covering Namespace as a Service, GitOps, the Orchestrator, and Event-Driven Ansible governance on OpenShift — acknowledges the use of artificial intelligence as a collaborative tool in the creation, structuring, and communication of its content.

This statement is offered in the interest of transparency and professional integrity. It affirms that AI collaboration within this project is governed by deliberate standards of care across three dimensions: **Creation Diligence**, **Transparency Diligence**, and **Deployment Diligence**.

This document does not serve as a disclaimer. It serves as a commitment — a reflection of how this project approaches AI-assisted work with the same rigour applied to all other aspects of its technical and educational philosophy.

-----

## What AI Was Used For

To be specific about where AI contributed in this project:

- **Documentation authoring** — Lab module pages (`.md` and `.adoc`) for Phase 1 (self-service namespace provisioning), Phase 2 (Orchestrator workflows), and Phase 3 (EDA governance) were drafted with AI assistance and reviewed by the author for technical accuracy.
- **Generic content extraction** — Existing lab files containing environment-specific variables (cluster URLs, credentials, namespace names) were processed with AI to produce reusable, environment-agnostic versions suitable for use across different repositories and workshop environments.
- **Architecture articulation** — The framing of the platform as a *“Continuous Delivery Engine for the Real World”* and the three-phase model (GitOps → Orchestrator → EDA) was developed collaboratively with AI to sharpen and communicate the conceptual structure of the work.
- **Code scaffolding** — EDA rulebook examples, Ansible playbook templates, Helm chart snippets, and SonataFlow workflow YAML were generated with AI assistance as illustrative starting points, then reviewed and adapted by the author.
- **Diligence and review** — The AI was used to cross-check technical claims, identify gaps in lab steps, and suggest additional content such as troubleshooting tables and architecture diagrams.

In all cases, the underlying technical work — the Decision Environment build manifest, the OpenShift RBAC design, the rulebook architecture, the credential model — was authored and validated by the human contributor. AI accelerated the communication of that work; it did not originate it.

-----

## Creation Diligence

Creation Diligence refers to the standards applied when AI is used to generate, draft, or structure workshop content.

All AI-assisted content produced within the scope of this project is subject to the following practices:

- **Human oversight at every stage.** No AI-generated content is treated as final or authoritative without review and validation by the author. Technical accuracy — particularly for OpenShift, AAP, EDA, and SonataFlow specifics — is verified by someone with direct hands-on experience of the subject matter.
- **Accuracy verified against source material and lived experience.** AI outputs are cross-referenced with Red Hat official documentation, upstream project documentation (ArgoCD, SonataFlow, juniper.eda.k8s), and the author’s own implementation experience. Where AI produced plausible but inaccurate technical detail, it was corrected before inclusion.
- **Content shaped by domain expertise.** AI was used to accelerate drafting and organisation — not to substitute for the subject matter knowledge required to design and operate a Namespace as a Service platform on OpenShift at enterprise scale.
- **Iterative refinement is the standard.** Workshop modules were reviewed across multiple passes to ensure alignment with the intended learning outcomes, technical accuracy, and the progression from Phase 1 through Phase 3.
- **Environment variables and credentials were deliberately removed.** A specific and intentional use of AI in this project was the systematic removal of hardcoded environment variables, cluster URLs, usernames, and passwords from lab content — making the material portable and safe for use in any environment.

-----

## Transparency Diligence

Transparency Diligence refers to the obligation to openly acknowledge AI’s role in this project and to ensure that all users of this workshop have an accurate understanding of how its content was produced.

This project is committed to the following transparency practices:

- **AI involvement is disclosed proactively.** This statement exists precisely to make that disclosure explicit — not buried in a footnote, but as a named document in the repository.
- **Users are not misled about authorship.** This workshop represents a collaboration between the human author and AI tools. The technical design, implementation decisions, and platform architecture are the author’s own. The documentation, lab structure, and written explanations benefited from AI assistance.
- **The limitations of AI are recognised.** AI systems can produce technically plausible but incorrect content, particularly in fast-moving domains like AAP 2.4+ EDA features, SonataFlow API changes, and OpenShift release-specific behaviour. All such content has been reviewed; however, users are encouraged to validate against the latest official documentation for their specific platform versions.
- **This statement is a living document.** As AI tools evolve and as further AI assistance is used in extending this workshop, this statement will be updated to reflect current practices.

-----

## Deployment Diligence

Deployment Diligence refers to the standards applied when AI-assisted content is published and used by others — ensuring that what reaches workshop participants has been responsibly validated and is fit for its educational purpose.

The following practices govern the deployment of AI-assisted content in this workshop:

- **Review precedes publication.** AI-assisted lab content does not reach this repository without passing through the author’s review — evaluating technical accuracy, step-by-step correctness, and alignment with the workshop’s learning objectives.
- **Code and YAML examples are illustrative starting points.** Rulebook YAML, playbook examples, Helm chart snippets, and SonataFlow workflow definitions in this workshop are intended to demonstrate patterns and concepts. They are not certified production configurations and should be adapted to your organisation’s specific environment, security posture, and requirements before deployment.
- **Credentials and secrets are never included.** A non-negotiable constraint applied throughout: no real tokens, passwords, API keys, or cluster-specific values appear anywhere in this workshop content. Placeholders (e.g. `<your-cluster-api-url>`, `<your-namespace>`) are used consistently.
- **Feedback is welcomed.** If you identify technical inaccuracies, outdated references, or content that does not work as described in your environment, please open an issue. AI-assisted content, like all content, benefits from community review and correction.
- **AI-assisted outputs are not deployed as authoritative compliance or security guidance.** Where this workshop touches on security posture (RBAC design, least-privilege access, credential management), the content represents recommended practice drawn from Red Hat documentation and the author’s experience — not a certified security audit or compliance determination.

-----

## Closing Reflection

This workshop exists to make a genuinely complex platform engineering pattern — self-governing namespaces with GitOps, Orchestrator, and Event-Driven Ansible — accessible to practitioners who want to build it.

AI collaboration made it possible to produce documentation at a pace and breadth that would have taken significantly longer alone. But the ideas, the architecture, the technical decisions, and the platform design are human work — built from real experience running these systems on real clusters.

That combination — human expertise amplified by AI tooling — is itself an illustration of the platform engineering philosophy this workshop teaches: **the right tool, doing the right job, with a human responsible for the outcome.**

-----

## AI Tool Used

|Tool              |Version          |Primary Use                                                                                                             |
|------------------|-----------------|------------------------------------------------------------------------------------------------------------------------|
|Claude (Anthropic)|Claude Sonnet 4.6|Documentation drafting, content restructuring, environment variable removal, architecture articulation, code scaffolding|

-----

*This statement was itself drafted with AI assistance and reviewed by the author.*