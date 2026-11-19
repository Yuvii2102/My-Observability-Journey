# Day 7 – PagerDuty and End-to-End Incident Management

## 1. Introduction

In this lesson, I learned about **PagerDuty**, an incident management platform widely used in DevOps and Site Reliability Engineering (SRE).

PagerDuty helps teams detect incidents, notify the right people, coordinate incident response and analyse what happened after an incident is resolved.

Instead of handling everything manually, PagerDuty automates most of the incident management process.

## 2. What is Incident Management?

Incident Management is the process of identifying, understanding, resolving and analysing unexpected problems that affect an application's availability or performance.

For example, imagine an e-commerce application where customers are unable to make payments.

This is a serious issue because customers cannot complete their transactions.

The team needs to:

1. Identify the problem.
2. Understand which service is affected.
3. Notify the responsible team.
4. Investigate the root cause.
5. Fix the issue.
6. Perform a post-incident analysis.

This complete process is called Incident Management.

## 3. Why is Manual Incident Management Difficult?

Let's understand this with an example.

Imagine I am working as a Site Reliability Engineer (SRE) in a company that provides online courses.

One of the most important services in the application is the **Payment Service**.

The company uses Grafana for monitoring and has configured an alert:

- If successful HTTP requests to the Payment Service fall below 99%, trigger an alert.
- Treat this alert as a P1 incident because payment failures directly affect customers and business transactions.

Now imagine this alert is triggered at 2 AM.

If the company does not use an incident management platform, the incident manager must handle everything manually.

### Challenges of manual incident management

**1. Collecting incident information**

The incident manager must find out:

- Which service is affected?
- Which API is failing?
- When did the problem start?
- How many customers are affected?

**2. Finding the right people**

The incident manager must identify the Subject Matter Expert (SME) and contact the required team members.

These may include:

- Developers
- Database Administrators (DBAs)
- QA Engineers
- SREs
- Kubernetes or infrastructure engineers

**3. Creating communication channels**

The incident manager may need to:

- Create a Slack channel.
- Create a Jira ticket.
- Add all the required team members.
- Share logs, documents and incident details.

**4. Managing the incident**

The incident manager must continuously coordinate the team, share updates, assign responsibilities and explain the current situation to newly added members.

**5. Performing post-incident analysis**

After resolving the issue, the team must identify the root cause and understand how to prevent the same problem in the future.

### Why is this a problem?

All these activities take time.

If the team spends 12–24 hours just identifying and coordinating the incident, the problem may become worse.

More customers may be affected, transactions may fail and the company may lose revenue.

This is why companies use automated incident management platforms.

## 4. What is PagerDuty?

PagerDuty is an end-to-end incident management platform that helps organisations detect incidents, notify the right teams, coordinate responses and perform post-incident analysis.

It integrates with monitoring tools such as Grafana and communication platforms such as Slack and Microsoft Teams.

For example, when Grafana detects an issue, it can send an alert to PagerDuty.

PagerDuty can then help the team:

- Detect and manage the incident.
- Notify the right people.
- Assign incident ownership.
- Create and manage incident workflows.
- Coordinate troubleshooting.
- Document important updates.
- Perform post-incident reviews.

The main goal is to reduce the time required to identify and resolve incidents.

## 5. How PagerDuty Works

Let's use the same Payment Service example.

Without PagerDuty, the incident manager performs most of the work manually.

With PagerDuty, the process becomes more automated.

### Basic workflow

1. Grafana monitors the Payment Service.
2. Grafana detects that successful HTTP requests have dropped below 99%.
3. Grafana sends an alert to PagerDuty.
4. PagerDuty creates an incident.
5. PagerDuty notifies the responsible people.
6. A Slack channel is used to coordinate the response.
7. The team investigates and resolves the issue.
8. PagerDuty helps generate a post-incident review.

### Architecture

```text
       Payment Service
              |
              v
           Grafana
       (Monitoring Tool)
              |
              v
          PagerDuty
      (Incident Management)
              |
              v
       Slack Integration
              |
      +-------+-------+
      |       |       |
      v       v       v
   Developer  SRE    DBA
      |       |       |
      +-------+-------+
              |
              v
       Incident Resolution
              |
              v
      Post-Incident Review
```

This integration helps teams manage incidents from detection to resolution.

## 6. Three Major Stages of Incident Management in PagerDuty

PagerDuty helps manage the incident lifecycle through three major stages.

### Stage 1: Incident Identification

The first stage is identifying and understanding the incident.

For example:

- Grafana detects an issue with the Payment Service.
- It sends an alert to PagerDuty.
- PagerDuty creates an incident.
- The responsible team members are notified.
- The incident is assigned to the appropriate person.

The team can acknowledge the incident and take ownership of it.

**Acknowledging an incident** means that someone has accepted responsibility for handling it.

The team can also add notes to the incident.

For example, an SRE might identify that the Payment Service is experiencing failures because of high memory usage in a virtual machine.

The Subject Matter Expert (SME) can review the information and confirm the possible cause.

Once the incident is understood, the team can trigger a workflow or script to collect additional information, such as database health or virtual machine status.

### Stage 2: Incident Response and Resolution

The second stage is investigating and fixing the issue.

PagerDuty provides incident response capabilities that help teams collaborate and investigate problems.

One feature discussed by my teacher is **PagerDuty Advance**, which can help summarise incident information and provide useful context.

For example, the team can ask about recent changes made to the application.

PagerDuty Advance can provide information such as:

- Recent deployments.
- Release notes.
- Changes made to the service.
- The person who committed a change.

This information helps the team investigate whether a recent deployment caused the problem.

The team can also investigate which customers are affected using relevant connected support information.

#### Adding new team members

Sometimes, the team discovers that the problem is related to the database.

In that situation, the incident manager can bring in a Database Administrator (DBA).

The newly added person needs to understand what has happened so far.

PagerDuty can summarise the incident and provide the current status.

This saves the team from explaining the entire incident again.

#### Resolving the incident

After investigating the problem, the team may decide to roll back the application to its previous version.

The team performs the rollback and checks whether the issue has been resolved.

This is how PagerDuty helps coordinate the incident response process.

### Stage 3: Post-Incident Analysis

The third stage begins after the incident has been resolved.

The team performs a post-incident review to understand:

- What happened?
- When did the incident start?
- Which services were affected?
- What was the root cause?
- How was the issue resolved?
- What can be improved to prevent it from happening again?

PagerDuty helps document the resolution and share the information with stakeholders.

It can also generate a post-incident review report.

This helps the organisation learn from incidents and improve its systems.

## 7. PagerDuty and Slack Integration

One important feature discussed in this lesson is PagerDuty's Slack integration.

PagerDuty provides a Slack-first incident response experience.

This means teams can manage many incident-related activities directly from Slack.

For example, when an incident occurs:

1. PagerDuty creates a Slack channel for the incident.
2. It adds the required team members.
3. It shares the incident details.
4. Team members can acknowledge the incident.
5. They can share updates and investigate the issue.
6. They can assign tasks and run workflows.
7. They can mark the incident as resolved.

PagerDuty also provides slash commands that can be used inside Slack.

For example, `/pd` commands can help users perform actions such as:

- Reassigning an incident.
- Running a workflow.
- Creating a task.
- Escalating an incident.

This makes communication and incident coordination easier.

## 8. Incident Priority: P1 and P2

PagerDuty allows teams to configure different incident workflows based on incident priority.

For example:

**P1 – Critical Incident**

A critical issue that needs immediate attention.

Example: The Payment Service is unavailable and customers cannot complete transactions.

The team may configure a workflow that immediately notifies the responsible engineers and starts the required response process.

**P2 – High-Priority Incident**

A serious issue that requires attention but may not have the same immediate business impact as a P1 incident.

The team can configure a different workflow for this priority.

The important point is that incident workflows can be customised based on the type and priority of the incident.

## 9. AI-Powered Incident Summarisation

Another feature discussed by my teacher is AI-powered incident summarisation.

Imagine a new DBA joins an ongoing incident.

Instead of asking the incident manager to explain everything from the beginning, the new team member can request an incident summary.

PagerDuty can provide information about:

- What happened.
- What the team has investigated.
- The suspected root cause.
- What actions have already been performed.
- The current incident status.

This helps new team members understand the situation quickly.

## 10. Other PagerDuty Features Discussed

My teacher also discussed some additional PagerDuty capabilities.

- **Incident Workflows:** Automate actions based on incident conditions and priorities.
- **Slack and Jira Integration:** Connect incident management with communication and issue tracking.
- **Post-Incident Reviews:** Document what happened and analyse the root cause.
- **Status Pages:** Share incident status with customers and subscribers.
- **Roles and Permissions:** Control who can access and manage incidents.
- **Security Incidents:** Configure specialised response processes for security-related incidents.
- **PagerDuty Advance:** Use AI-powered capabilities to help summarise incidents and support investigations.

The lesson also mentioned that some features and usage limits vary between Professional and Business plans.

## 11. What I Learned

Through this lesson, I learned:

1. What incident management is and why it is important in DevOps.
2. Why handling incidents manually is time-consuming.
3. What PagerDuty is and how it helps automate incident management.
4. How Grafana can send monitoring alerts to PagerDuty.
5. How PagerDuty integrates with Slack.
6. How teams acknowledge, assign and manage incidents.
7. How incident priority such as P1 and P2 can be used to configure workflows.
8. How PagerDuty Advance helps summarise incident information.
9. How teams coordinate incident response and resolve issues.
10. Why post-incident analysis and root cause analysis are important.

## 12. Final Summary

In this lesson, I understood how PagerDuty helps organisations manage incidents from detection to resolution.

I learned that incident management is not just about fixing an issue. It also involves identifying the problem, notifying the right people, coordinating the response, documenting the process and analysing the root cause.

PagerDuty automates many of these activities through integrations with monitoring tools and communication platforms.

**The main takeaway:** PagerDuty helps DevOps and SRE teams respond to incidents faster, improve collaboration and reduce the impact of failures on customers.

**Day 7 – PagerDuty and End-to-End Incident Management.**
