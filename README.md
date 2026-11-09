# RaceDay API — PROG 6212 POE Part 2

ASP.NET Core Web API, C#, Entity Framework Core, SQL Server, cookie authentication, role-based authorisation, Swagger, unit/integration test starter, and GitHub Actions.

## Requirements
- Visual Studio 2022
- .NET 8 SDK
- SQL Server reachable at `localhost`
- SQL Server Management Studio (optional, for checking the database)

## Important schema note
This project follows the eight entities in the supplied RaceDay Part 1 ERD:
`USERS`, `EVENTS`, `CATEGORIES`, `EVENT_ROUTES`, `EVENT_WEATHER`,
`EVENT_STATUS_HISTORY`, `ENROLMENTS`, and `RESULTS`.

The ERD image does not specify every SQL type/length/constraint or all optionality rules. Review the model mapping against the actual Part 1 SQL script before applying a migration to a database containing important data. The default connection string targets a separate database called `RaceDayPart2` to avoid changing your original Part 1 database.

## 1. Open the project
Open `RaceDay.sln` in Visual Studio. Restore NuGet packages.

## 2. Check SQL Server
The default connection string is:
`Server=localhost;Database=RaceDayPart2;Trusted_Connection=True;TrustServerCertificate=True;`

If your local SQL Server uses SQL authentication or a named instance, update `RaceDay.API/appsettings.json`.

## 3. Create the database with EF Core
Open **Tools > NuGet Package Manager > Package Manager Console**.
Set **Default project** to `RaceDay.API`, then run:

```powershell
Add-Migration InitialRaceDayCreate
Update-Database
```

If `Add-Migration` is unavailable, run:
```powershell
dotnet tool install --global dotnet-ef --version 8.0.22
```
Restart Visual Studio and restore packages.

## 4. Run and open Swagger
1. In Visual Studio, open `RaceDay.sln`.
2. Right-click `RaceDay.API` and choose **Set as Startup Project**.
3. Choose the `RaceDay.API` launch profile (not IIS Express if you want the listed ports).
4. Build the solution (**Build > Build Solution**).
5. Open **Tools > NuGet Package Manager > Package Manager Console**, select `RaceDay.API` as Default project, and run `Update-Database` after creating the migration in step 3.
6. Press **F5**. Swagger should open at `https://localhost:65279/swagger` (or use the URL shown in Visual Studio).

Swagger is enabled in the supplied Development launch profile. Use the HTTPS URL so the secure authentication cookie works. After calling `POST /api/auth/login` successfully in Swagger, continue using the same browser tab; the browser should retain and send the HttpOnly cookie for protected requests.

## 5. Main endpoints
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/me`
- `GET /api/events`
- `GET /api/events/{id}`
- `POST /api/events` (Organiser)
- `PUT /api/events/{id}` (Organiser who owns the event)
- `DELETE /api/events/{id}` (Organiser who owns the event)
- `GET /api/events/{eventId}/categories`
- `POST /api/events/{eventId}/categories` (Organiser who owns the event)
- `POST /api/enrolments` (Participant)
- `GET /api/enrolments/my` (Participant)
- `GET /api/events/{eventId}/enrolments` (Organiser who owns the event)
- `POST /api/results` (Organiser who owns the event)
- `GET /api/results/my` (Participant)
- `GET /api/events/{eventId}/routes`
- `GET /api/events/{eventId}/weather`
- `GET /api/events/{eventId}/status-history`

## Authentication and role notes
Registration accepts only `Organiser` or `Participant`. Passwords are stored as ASP.NET Core password hashes, never as plain text. Successful login issues an HttpOnly authentication cookie. Swagger requests can use the cookie after login if the browser permits it. Do not deploy this development configuration publicly without reviewing HTTPS, cookie SameSite/CSRF protection, account verification, rate limiting, and secret management.

## Improvements included
- Password policy extracted into a testable helper.
- Duplicate email registration is handled defensively, with a unique database index as the final safeguard.
- Unique composite index prevents duplicate enrolments for the same participant/category.
- Event type is constrained to `run`, `walk`, or `cycle` at the database level.
- Organisers can add/update an event route and add weather forecasts for events they own.
- Controllers use role checks and verify event ownership for event-management operations.

## Known items to verify before submission
- Compare column types, nullability, string lengths, and the `EVENT_ROUTES` unique `EventID` rule with your original Part 1 SQL script. Your ERD labels `EventID` as FK, UK in `EVENT_ROUTES`, so this implementation models one route per event.
- Add integration tests for API status codes and role access using `WebApplicationFactory` before claiming comprehensive endpoint coverage.
- Configure anti-forgery/CSRF protection and secure cookie settings appropriate to your final hosting environment.
- The public registration endpoint allows a user to select either role, as the assignment describes; in a real production system, Organiser privileges should normally require approval or an invitation.

## Tests
The `RaceDay.Tests` project contains basic model/service tests and can be run with:
```powershell
dotnet test
```
The GitHub Actions workflow builds and tests on pushes and pull requests.
