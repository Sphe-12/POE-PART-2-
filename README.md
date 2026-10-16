# RaceDay API — Part 2

ASP.NET Core 8 Web API based on the supplied Part 1 `RaceDayDB` schema: `Roles`, `Users`, `Events`, `Categories`, `EventEnrollments`, and `Results`.

## Requirements
- .NET 8 SDK
- SQL Server or SQL Server LocalDB
- Visual Studio 2022 or VS Code

## Database setup
1. Open the Part 1 SQL script in SQL Server Management Studio.
2. Execute it to create/populate `RaceDayDB`.
3. If using SQL Server Express or another instance, update `ConnectionStrings:RaceDay` in `RaceDay.Api/appsettings.json`.

The EF Core model maps to the existing schema. Role records are seeded with IDs 1–3. Do not run a destructive reset script against a database containing data.

## Run
From the solution folder:
```bash
dotnet restore RaceDay.Api/RaceDay.Api.csproj
dotnet run --project RaceDay.Api/RaceDay.Api.csproj
```
Open the Swagger URL printed in the terminal (the launch profile uses `http://localhost:5080/swagger`).

## Authentication/session
1. `POST /api/auth/register` with `firstName`, `lastName`, `email`, `password`, `role` (`Organizer` or `Participant`), and optional `phoneNumber`.
2. `POST /api/auth/login` with email and password. Keep the `.RaceDay.Session` cookie returned by the API; Swagger/browser requests in the same browser session can reuse it.
3. `POST /api/auth/logout` clears the session.
Passwords are hashed using ASP.NET Core's `PasswordHasher`; raw passwords are never saved. Public registration deliberately does not allow `Administrator`, preventing privilege escalation. Session state expires after 30 minutes.

## Endpoints
| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/api/auth/register` | Public | Register Organizer/Participant |
| POST | `/api/auth/login` | Public | Create authenticated session |
| POST | `/api/auth/logout` | Session | End session |
| GET | `/api/auth/me` | Session | Current user |
| GET/PUT | `/api/profile/me` | Authenticated | View/update own profile |
| GET | `/api/events` | Public | List events |
| GET | `/api/events/{id}` | Public | Event details |
| POST | `/api/events` | Organizer | Create event |
| PUT/DELETE | `/api/events/{id}` | Owning Organizer | Update/delete own event |
| GET | `/api/events/{eventId}/categories` | Public | View event categories |
| POST | `/api/events/{eventId}/categories` | Owning Organizer | Add category |
| POST | `/api/enrollments` | Participant | Enrol in a category |
| GET | `/api/enrollments/mine` | Participant | View own enrolments |
| GET | `/api/enrollments/event/{eventId}` | Owning Organizer | View enrolments for own event |
| POST | `/api/enrollments/{enrollmentId}/result` | Owning Organizer | Record/update result |
| GET | `/api/results/mine` | Participant | View own results |

`EventRequest` accepts `eventName`, `description`, `eventDate`, `location`, and optional `status` (`Upcoming`, `Open`, `Closed`, `Completed`, `Cancelled`). `CategoryRequest` accepts `categoryName`, `distanceKM`, `minimumAge`, `maximumAge`, and `entryFee`. `ResultRequest` accepts `finishTimeSeconds`, `position`, and `resultStatus` (`Finished`, `DNF`, `Disqualified`).

## Tests
```bash
dotnet test RaceDay.Tests/RaceDay.Tests.csproj
```
The included tests verify password hashing/verification and the public role-registration rule. Add integration tests with a disposable SQL Server or EF Core test provider to exercise controller authorization and persistence.

## GitHub Actions
The workflow at `.github/workflows/dotnet.yml` restores, builds, and tests on every push and pull request. After pushing, open the repository's **Actions** tab and capture a screenshot of the successful green run for this README. A screenshot cannot be truthfully supplied until the workflow has run in your own GitHub repository.

## Required submission items still to do
- Push to your own GitHub repository and make at least 20 meaningful commits (this project cannot fabricate genuine commit history).
- Add your actual successful GitHub Actions screenshot.
- Record and add an unlisted YouTube demonstration link.
