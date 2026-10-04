<div align="center">

<img src="assets/mockups/hero.png" alt="MijnDakje app screens" width="100%">

# MijnDakje

**A Dutch rental-housing platform that scrapes broker sites, scores every new listing against your search profile and applies on your behalf.**

![.NET](https://img.shields.io/badge/.NET-9-512BD4?logo=dotnet&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind-v4-06B6D4?logo=tailwindcss&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-11.8-003545?logo=mariadb&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-Billing-635BFF?logo=stripe&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-compose-2496ED?logo=docker&logoColor=white)

</div>

> [!NOTE]
> This is a **showcase repository**. The production source code is private. Everything here
> (screens, mock-ups, diagrams) is generated from the real frontend running against **dummy data**.
> Names, listings, addresses and figures are fictional.

---

## The app in motion

<div align="center">
  <img src="assets/demo.gif" alt="Walkthrough" width="320">
  <br><sub>The real Next.js build on a phone viewport: dashboard, matches, listings, search profiles.</sub>
</div>

---

## What it does

Finding a rental in the Netherlands means refreshing a dozen broker sites and reacting within minutes.
MijnDakje scrapes those sites continuously, scores each new listing against weighted search criteria,
alerts the renter and, for Pro users, submits the application automatically.

<img src="assets/mockups/features.png" alt="Features" width="100%">

| For renters | For admins |
|---|---|
| **Search profiles**: location radius, rent, size, rooms, energy label, pets and more, each with a weight and a floor | **Platform dashboard**: users, subscriptions, MRR, match activity, scraper health |
| **Weighted match scores**: every listing gets a score with a per-criterion breakdown | **Broker management**: enable, pause, trigger and inspect scrape runs per broker |
| **Auto-apply (Pro)**: apply automatically above a score threshold, or approve manually | **User management**: roles, subscriptions, profiles, matches |
| **Mailbox**: broker replies arrive in a personal inbox | **System settings**: registration, e-mail types, auto-apply switch |
| **Notifications**: e-mail, push and a notification buddy who gets the alerts too | **Affiliates**: sign-up, review, commissions |
| **Billing**: Basic and Pro tiers via Stripe, referral credit | |

### Admin console

<img src="assets/mockups/admin.png" alt="Admin" width="100%">

### Languages

<img src="assets/mockups/languages.png" alt="Languages" width="100%">

---

## Architecture

A Next.js frontend proxies to an ASP.NET Core API built in Clean Architecture layers. Scraping, matching
and applying run in a separate Hangfire worker so the API stays responsive.

```mermaid
flowchart LR
    subgraph sys["MijnDakje system"]
        direction LR
        web["Web app<br/>Next.js 16, React 19"]
        api["API<br/>ASP.NET Core, Identity"]
        core["Application / Domain<br/>matching, subscriptions"]
        jobs["Jobs host<br/>Hangfire, Playwright"]
        db[("MariaDB<br/>app data + Hangfire")]
    end
    subgraph ext["External systems"]
        brokers["Rental broker sites<br/>scrape + apply"]
        stripe["Stripe<br/>billing"]
        geo["Nominatim + Overpass<br/>geocoding"]
        mail["SMTP / IMAP / Web Push / PushOver<br/>notifications + mailbox"]
    end
    web --> api --> core --> db
    jobs --> core
    jobs --> db
    jobs -.-> brokers
    api -.-> stripe
    api -.-> geo
    jobs -.-> mail
```

### Domain model (simplified)

```mermaid
erDiagram
    USER ||--o| PROFILE : has
    USER ||--o{ SEARCH_PROFILE : defines
    USER ||--o| SUBSCRIPTION : pays
    USER ||--o{ MATCH : receives
    USER ||--o{ USER_MESSAGE : reads
    BROKER ||--o{ LISTING : publishes
    BROKER ||--o{ SCRAPE_RUN : "scraped by"
    SCRAPE_RUN ||--o{ SCRAPE_RUN_LOG_ENTRY : logs
    LISTING ||--o{ MATCH : "scored as"
    SEARCH_PROFILE ||--o{ MATCH : produces
    USER ||--o{ REFERRAL : refers
    AFFILIATE ||--o{ AFFILIATE_COMMISSION : earns
```

---

## Tech stack

| Layer | Technology |
|---|---|
| **Client** | Next.js 16 (App Router), React 19, TypeScript, Tailwind v4, shadcn/ui, Recharts, Leaflet, next-intl (nl / en) |
| **Backend** | .NET 9, ASP.NET Core, Clean Architecture (Domain, Application, Infrastructure, API, Jobs) |
| **Auth** | ASP.NET Core Identity, cookie session, 2FA |
| **Data** | MariaDB 11.8, EF Core (Pomelo) with migrations |
| **Background work** | Hangfire (MySQL storage) with recurring scrape, match, apply and cleanup jobs |
| **Integrations** | Stripe, Playwright broker scrapers, Nominatim, Overpass, MailKit (SMTP/IMAP), Web Push, PushOver |
| **Observability** | OpenTelemetry (Prometheus exporter), health checks |
| **Local dev** | .NET Aspire AppHost, Docker Compose |
| **Testing** | xUnit + Moq (backend, 470+ test methods), Vitest + Testing Library (frontend, 160+ cases), Playwright end-to-end suite |

---

## Engineering highlights

- **Strategy pattern per broker**: 29 broker strategies behind a resolver, with separate list, detail and apply flows, so a new broker is one class.
- **Weighted scoring with floors**: every criterion has a target, a floor and a weight; the UI shows the per-criterion breakdown behind each score.
- **Jobs isolated from the API**: scraping, matching, applying, mailbox polling and cleanup run in their own Hangfire host.
- **Idempotent billing**: Stripe webhook events are recorded so a replayed event is not processed twice.
- **Tier enforcement in the domain**: Basic and Pro limits (search profiles, auto-apply eligibility) are enforced server-side.
- **Security reviewed**: the repo carries a pentest findings document and security headers on the frontend.

---

## Delivery

Each service ships as a Docker image (API, Jobs, frontend) and the full stack runs with Docker Compose.

```mermaid
flowchart LR
    dev["Developer<br/>.NET + Next.js"] --> img["Docker images<br/>API, Jobs, Web"] --> run["Docker Compose<br/>MariaDB, Nominatim, Overpass"]
```

---

## Screens

<table>
  <tr>
    <td><img src="assets/screens/d-home.png" width="420"></td>
    <td><img src="assets/screens/d-matches.png" width="420"></td>
  </tr>
  <tr>
    <td align="center"><sub>Dashboard</sub></td>
    <td align="center"><sub>Matches</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/d-listings.png" width="420"></td>
    <td><img src="assets/screens/d-profiles.png" width="420"></td>
  </tr>
  <tr>
    <td align="center"><sub>All listings</sub></td>
    <td align="center"><sub>Search profiles</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/d-messages.png" width="420"></td>
    <td><img src="assets/screens/d-admin-home.png" width="420"></td>
  </tr>
  <tr>
    <td align="center"><sub>Mailbox</sub></td>
    <td align="center"><sub>Admin dashboard</sub></td>
  </tr>
  <tr>
    <td colspan="2"><img src="assets/screens/d-admin-brokers.png" width="860"></td>
  </tr>
  <tr>
    <td colspan="2" align="center"><sub>Admin: brokers</sub></td>
  </tr>
</table>

---

<div align="center">

Made by **[Mennova](https://mennova.nl)** · more projects on **[mennova.nl/cases](https://mennova.nl/cases)**

</div>
