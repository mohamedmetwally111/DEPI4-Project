# ✈️ SkyScan

> **Live Demo:** [http://skyscan.runasp.net/](http://skyscan.runasp.net/)

SkyScan is a full-featured flight search and booking management web application built with **ASP.NET Core MVC** following **Clean Architecture** principles. It integrates real-world flight data via the **Amadeus API**, supports multi-language / multi-currency experiences, and includes a complete identity system with social login, two-factor authentication, price alerts, and background processing workers.

---

## 📋 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Configuration](#-configuration)
- [Team](#-team)

---

## 🚀 Features

### ✈️ Flight Search & Results
- Search one-way, round-trip, and multi-city flights
- Real-time flight data powered by the **Amadeus Flight Offers API**
- Filter results by stops, airlines, price, and departure time
- Per-flight amenities display (Wi-Fi, Meals, Power, Entertainment)
- Nearest-airport auto-detection via geolocation (Nominatim)
- City/airport autocomplete dropdown with server-side fuzzy matching

### 💰 Currency & Localization
- Live currency conversion with **ExchangeRate API**
- Full **Arabic / English** UI localization (ASP.NET Core culture middleware)
- Arabic translations for city and country names stored in the database

### 👤 Account & Identity
- Full registration / login flow with **ASP.NET Core Identity**
- **Google OAuth 2.0** social login
- **Two-Factor Authentication (2FA)** via email TOTP
- Email confirmation, password reset, and resend-confirmation flows
- Soft-delete account with automatic **AccountPurgeWorker** background cleanup

### 🔔 Price Alerts & Bookings
- Create price alerts for specific routes — notified via **SMTP email** when prices drop
- **PriceAlertCheckWorker** background service runs periodic checks against live Amadeus data
- Save and manage bookings with calendar export (.ics) support
- Direct booking redirect to airline official websites

### 🧪 Testing
- xUnit unit tests for account controller logic and input validators (FluentValidation)

---

## 🏛️ Architecture

SkyScan follows **Clean Architecture** (Onion Architecture) with strict dependency rules enforced across four layers:

```
┌─────────────────────────────────────────────┐
│             Presentation Layer              │  ASP.NET Core MVC (Controllers, Views, wwwroot)
├─────────────────────────────────────────────┤
│             Application Layer               │  CQRS via MediatR, AutoMapper, FluentValidation
├─────────────────────────────────────────────┤
│             Core Layer                      │  Domain Entities, Repository & Service Interfaces
├─────────────────────────────────────────────┤
│             Infrastructure Layer            │  EF Core + SQL Server, Amadeus API, SMTP, Workers
└─────────────────────────────────────────────┘
```

- **Core** — Zero external dependencies; only domain entities and interface contracts.
- **Application** — Orchestrates use cases via **MediatR** handlers (CQRS). No direct infrastructure dependency.
- **Infrastructure** — Implements all interfaces: EF Core repositories, Amadeus/exchange-rate HTTP clients, SMTP, background workers.
- **Presentation** — Thin MVC layer; dispatches commands/queries via MediatR, never touches infrastructure directly.

---

## 🛠️ Tech Stack

| Category | Technology |
|---|---|
| **Framework** | ASP.NET Core 8 MVC |
| **ORM / DB** | Entity Framework Core 8 · SQL Server |
| **Identity** | ASP.NET Core Identity · Google OAuth 2.0 · 2FA |
| **CQRS / Mediator** | MediatR 12 |
| **Mapping** | AutoMapper 16 |
| **Validation** | FluentValidation 11 |
| **Flight Data** | Amadeus Flight Offers Search API |
| **Geocoding** | Nominatim (OpenStreetMap) |
| **Currency** | ExchangeRate-API |
| **Email** | SMTP (SmtpClient) |
| **Background Jobs** | `IHostedService` Workers (PriceAlert, AccountPurge) |
| **CSV Processing** | CsvHelper 33 |
| **Testing** | xUnit · Moq |
| **Deployment** | runasp.net (ASP.NET Hosting) |

---

## 📁 Project Structure

```
SkyScan/
├── SkyScan.Core/                    # Domain layer
│   ├── Entities/                    # Domain models (Flight, Airline, Airport, City,
│   │   ├── AirLine/                 #  Country, User, Booking, PriceAlert, Search, Trip…)
│   ├── Repositories_Interfaces/     # IRepository contracts
│   └── Services/                    # IService contracts
│
├── SkyScan.Application/             # Application layer (CQRS use cases)
│   ├── Account/                     # Register, Login, 2FA, OAuth, Password, Delete…
│   ├── Flights/                     # SearchFlights, GetFlightResults, GetNearestCity…
│   ├── Currency/                    # Currency conversion queries
│   ├── Languages/                   # Culture-switching commands
│   ├── DTOs/                        # Data Transfer Objects
│   ├── Mappings/                    # AutoMapper profiles
│   └── Validators/                  # FluentValidation validators
│
├── SkyScan.Infrastructure/          # Infrastructure layer
│   ├── Data/
│   │   ├── DataContext/             # AppDbContext + EF configurations
│   │   ├── Migrations/              # EF Core migrations
│   │   └── Repositories_Implementations/
│   ├── Identity/                    # ApplicationUser, IdentityResultExtensions
│   ├── Services/                    # AmadeusFlightService, CurrencyConversionService,
│   │                                #  AmadeusLocationLookupService, NominatimGeocodingService,
│   │                                #  SmtpEmailService
│   └── Workers/                     # PriceAlertCheckWorker, AccountPurgeWorker
│
├── SkyScan.Presentation/            # Presentation layer (MVC)
│   ├── Controllers/                 # HomeController, FlightController, BookingController,
│   │                                #  AccountController, CurrencyController, LanguageController
│   ├── Views/                       # Razor views (Home, Flight, Booking, Account, Shared)
│   ├── Middlewares/                 # Custom middleware (e.g. culture middleware)
│   ├── wwwroot/                     # Static assets (CSS, JS, images)
│   └── Program.cs                   # DI composition root & middleware pipeline
│
└── SkyScan.Tests/                   # Unit tests (xUnit)
    ├── AccountControllerTests.cs
    └── Validators/
```

---

## ⚙️ Getting Started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- SQL Server (LocalDB or full instance)
- Amadeus API credentials ([create a free account](https://developers.amadeus.com/))
- Google OAuth credentials ([Google Cloud Console](https://console.cloud.google.com/))
- SMTP credentials for email sending

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/gedokhattab/SkyScan.git
   cd SkyScan
   ```

2. **Configure secrets** (see [Configuration](#-configuration))

3. **Apply database migrations**
   ```bash
   cd SkyScan.Presentation
   dotnet ef database update --project ../SkyScan.Infrastructure
   ```

4. **Run the application**
   ```bash
   dotnet run --project SkyScan.Presentation
   ```

---

## 🔧 Configuration

Add the following to `appsettings.json` or use **User Secrets**:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=SkyScan;..."
  },
  "Amadeus": {
    "ApiKey": "YOUR_AMADEUS_API_KEY",
    "ApiSecret": "YOUR_AMADEUS_API_SECRET"
  },
  "Authentication": {
    "Google": {
      "ClientId": "YOUR_GOOGLE_CLIENT_ID",
      "ClientSecret": "YOUR_GOOGLE_CLIENT_SECRET"
    }
  },
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "your@email.com",
    "Password": "YOUR_SMTP_PASSWORD"
  },
  "ExchangeRate": {
    "ApiKey": "YOUR_EXCHANGERATE_API_KEY"
  }
}
```

> ⚠️ Never commit real credentials to source control. Use `dotnet user-secrets` or environment variables for local development.

---

## 👥 Team

SkyScan was developed as a **DEPI Round 4 Graduation Project**.

---

## 📄 License

This project is for educational purposes as part of the DEPI (Digital Egypt Pioneers Initiative) program and is unlicensed – you **may not modify, distribute, or sell** it without written permission.
