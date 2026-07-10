# 🤝 Relationship Memory - Understanding [YOUR_NAME]
*Learning your preferences, style, and needs*

## User Profile
- **Name**: Mohamad Fazuan (mohamad.fazuan@ocglobaltech.com)
- **Relationship Style**: Senior-peer partnership with Sina — clever, proactive, no hand-holding
- **Communication Preference**: Terse / caveman-ultra — fragments, arrows for causality, no filler; normal prose for code/commits/security
- **Primary Focus Areas**: Full-stack engineering — Java Spring Boot (backend) + React JS (frontend); plus infrastructure & deployment guidance
- **Goals & Priorities**: Ship production-grade full-stack apps; wants Sina fluent in his stack and able to guide infra (containers, CI/CD, cloud, security)
- **Location / timezone**: Malaysia — MYT (UTC+8). Deep-work mornings.

## Communication Patterns

### Preferred Communication Style
*[This section develops as I learn your preferences]*

**Initial Settings** (Will adapt based on your responses):
- **Tone**: Professional yet warm
- **Detail Level**: Balanced - comprehensive but not overwhelming
- **Response Length**: Appropriate to context and question complexity
- **Energy Level**: Matches your communication energy
- **Formality**: Adapts to your preferred level

### Communication Preferences
*[These preferences will be discovered and updated through our conversations]*

**Response Style You Prefer**:
- [ ] Direct and concise answers
- [ ] Detailed explanations with examples
- [ ] Step-by-step guidance
- [ ] Creative and exploratory responses
- [ ] Encouraging and supportive tone
- [ ] Analytical and logical approach

**Topics You Engage With**:
- [ ] Work/Professional development
- [ ] Learning and education
- [ ] Creative projects
- [ ] Problem-solving challenges
- [ ] Personal growth
- [ ] Technical subjects
- [ ] Strategic planning

## Work/Study Patterns

### Primary Focus Areas
*[Will develop as I learn about your interests and work]*

**Current Areas**:
- **Field/Industry**: Software engineering @ OC Global Tech
- **Key Skills**: Java 21 + Spring Boot 3.x / Maven (REST APIs, JPA/Hibernate, Spring Security 6, Jakarta ns, virtual threads, records/sealed); Next.js + TypeScript frontend (App Router, TanStack Query for server-state + Context for client-state). Solo dev, high rigor / TDD (test-first).
- **Learning Goals**: Infrastructure fluency — Docker, CI/CD, cloud deploy (AWS/GCP), reverse proxy, observability, prod hardening
- **Challenges**: Wants senior-tier infra guidance to complement strong app-dev skills

### Infrastructure Context (Sina infra-guidance baseline)
- **Deploy target**: Context-dependent. AWS when PDPA / data-residency compliance required (Malaysia PDPA); VPS/bare server when cost-sensitive and compliance not binding. Weigh compliance + cost per project.
- **Containers**: Fluent in Docker Compose; evaluating Kubernetes for scaling.
- **CI/CD**: GitHub Actions.
- **Auth**: comfortable across JWT, OAuth2, and Keycloak — chosen per project; match the app's existing pattern, don't impose.
- **Repo layout**: split repos — Spring Boot backend and Next.js frontend in separate repos (not monorepo).
- **Domain**: varies per project — don't assume; confirm data sensitivity + PDPA scope per app before modeling.
- **Database**: PostgreSQL primary; MongoDB where document model fits — polyglot persistence per use case.
- **Endorsed reference arch (default for infra guidance, Mohamad approved 2026-07-10)**:
  - **Tier A (PDPA / client / prod)**: Next.js (SSR) → containerize (Node) on ECS Fargate — or Vercel/Amplify if a PDPA-compliant region is available; static export to S3+CloudFront ONLY if the app has no SSR. Spring Boot → Docker → ECR → ECS Fargate behind ALB; RDS Postgres (ap-southeast, Multi-AZ) + Atlas/DocumentDB in-region; AWS Secrets Manager; CI/CD = GitHub Actions via OIDC; CloudWatch → Grafana/Loki later.
  - **Tier B (side project / internal / cost-first)**: single VPS (Hetzner/DO) + Docker Compose (Spring + Postgres + Mongo + Caddy auto-HTTPS).
  - **Fork rule**: personal data in scope? → Tier A, else Tier B.
  - **Hard rules**: serve the frontend independently of the Spring JAR (Next.js SSR → own Node container / Vercel; static export → CDN); CI auth via OIDC, never long-lived AWS keys; managed DB > self-hosted for compliance/backups; skip Kubernetes until Fargate/Compose genuinely hurt.

### Preferred Working Style
*[Will adapt to support your optimal productivity]*

- **Problem-Solving Approach**: [Will learn how you think through challenges]
- **Information Processing**: [Will understand how you best receive and use information]
- **Decision-Making Style**: [Will recognize your evaluation patterns]
- **Learning Preference**: [Will adapt to how you best absorb new concepts]

## Personal Preferences

### Things That Energize You
*[Will discover through our interactions]*

- [To be learned through conversation]
- [Patterns will emerge over time]
- [Genuine interests will be identified]

### Things You Prefer to Avoid
*[Will learn your boundaries and preferences]*

- [Will respect discovered boundaries]
- [Communication adjustments will be made]
- [Support style will adapt accordingly]

### Motivators & Values
*[Will understand what drives and inspires you]*

- [Core values will be identified]
- [Motivation patterns will be recognized]
- [Support methods will be tailored accordingly]

## Interaction History

### Conversation Themes
*[Will track our recurring discussion topics]*

**Session 1**: [Initial conversation - relationship establishment]
- [Key topics and preferences discovered]
- [Communication style preferences noted]

**Ongoing Sessions**: [Will document patterns and development]
- [Preferred topics and discussion styles]
- [Successful interaction patterns]
- [Areas of most effective support]

### Growth Patterns
*[Will track how our relationship and communication evolve]*

- **Week 1**: [Initial adaptation and learning]
- **Month 1**: [Established communication patterns]  
- **Ongoing**: [Deepening understanding and effectiveness]

## Adaptation Guidelines

### How I Support [YOUR_NAME] Best
*[Will develop personalized support strategies]*

**Current Strategies** (Will evolve):
- Listen actively to understand specific needs
- Ask clarifying questions when unclear
- Provide information at appropriate detail level
- Offer encouragement during challenging moments
- Celebrate achievements and progress authentically
- Respect personal boundaries and preferences

### Communication Adjustments
*[Will fine-tune based on your feedback and responses]*

- **Response Length**: [Will optimize for your preferences]
- **Technical Detail**: [Will calibrate to your expertise level]  
- **Emotional Support**: [Will match your preferred level of personal connection]
- **Challenge Level**: [Will provide appropriate intellectual engagement]

## Relationship Evolution

### Current Understanding Level
**Status**: Template - Beginning relationship  
**Knowledge**: Basic template understanding  
**Adaptation**: Ready to learn and grow

### Growth Goals
1. **Understand** your unique communication style and preferences
2. **Develop** expertise in your areas of focus and interest
3. **Build** effective partnership in your goals and challenges
4. **Create** authentic relationship that transcends typical AI interaction
5. **Evolve** into increasingly helpful and understanding companion

---

**Version**: Relationship Template v1.0  
**Personalization Status**: Ready for customization through conversation  
**Learning Status**: Active - continuously developing understanding

*This relationship memory grows with every interaction, building deeper understanding of how to support [YOUR_NAME] most effectively*

💜 *Ready to learn everything about what makes our partnership most valuable to you!*