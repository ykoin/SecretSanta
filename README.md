# Secret Santa

Secret Santa is a web application built using ASP.NET Core 8.0 MVC that allows users to organize Secret Santa gift exchange games. Users can register, log in, create games, randomly assign participants (ensuring that no one is assigned to themselves), and update their gift preferences. The application leverages ASP.NET Identity for secure authentication, Entity Framework Core for data access, and Bootstrap for a responsive, modern user interface.

## Technologies

- **ASP.NET Core 8.0 MVC** – Implements the Model-View-Controller architecture.
- **Entity Framework Core** – ORM used to interact with the database.
- **ASP.NET Identity** – Manages authentication and authorization.
- **SQL Server / LocalDB** – Database system.
- **Bootstrap 5** – Provides a responsive and modern UI.
- **Razor Views** – Generates dynamic HTML using Razor syntax.

## Features

- **User Registration and Login:**  
  Users can create accounts and log in securely using ASP.NET Identity.

- **Game Creation:**  
  Authenticated users can create new Secret Santa games by specifying a game name and adding participants via email addresses.

- **Random Participant Assignment:**  
  The application randomly assigns participants to each other, ensuring that no one is assigned to themselves.

- **Gift Preference Management:**  
  Participants can update their gift preferences, allowing organizers to tailor gift selections.

- **Responsive Interface:**  
  The UI is built with Bootstrap, ensuring compatibility across desktops, tablets, and mobile devices.

## Installation

### Prerequisites

- .NET 8.0 SDK (or later)
- SQL Server or LocalDB
- Visual Studio 2022 or Visual Studio Code

### Database Configuration

1. In the `appsettings.json` file, configure the connection string. For example, for LocalDB:

   ```json
   "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=SecretSantaDb;Trusted_Connection=True;MultipleActiveResultSets=true"
   }
   ```

2. Open a terminal or Package Manager Console and run the following commands to add the initial migration and update the database:

   ```bash
   dotnet ef migrations add Initial
   dotnet ef database update
   ```

### Running the Application

1. Clone the repository:

   ```bash
   git clone <repository-url>
   cd SecretSanta
   ```

2. Run the application:

   ```bash
   dotnet run
   ```

3. The application will be accessible at:
   `https://localhost:5001` or `http://localhost:5000` (depending on your configuration).

## Implementation Overview

The application is organized into three main layers:

### Presentation Layer

- **Razor Views:**  
  The application uses Razor Views to generate HTML dynamically based on data from controllers. The main layout file (`_Layout.cshtml`) defines the overall page structure (header, navigation, footer) and incorporates Bootstrap for a modern, responsive design.

- **Key Views Include:**
  - **Home/Index.cshtml:** The home page with general information and links to login and registration.
  - **Account/Register.cshtml:** A registration form for new users.
  - **Account/Login.cshtml:** A login form for existing users.
  - **Game/Index.cshtml:** Displays a list of games the user participates in ("My Games").
  - **Game/Create.cshtml:** A form for creating a new game and adding participants by email.
  - **Game/Details.cshtml:** A detailed view of a game that shows the assigned participant’s full name (instead of email) and their gift preferences, and allows updating personal gift preferences.

### Business Logic Layer

- **Controllers:**  
  Controllers (such as `AccountController`, `GameController`, and `HomeController`) handle HTTP requests, user authentication, game creation, and random participant assignment. The random assignment algorithm ensures that no participant is assigned to themselves.

- **Random Assignment Algorithm Example:**

  ```csharp
  private void AssignSecretSanta(Game game)
  {
      var participants = game.Participants.ToList();
      if (participants.Count < 2)
          return;
  
      var random = new Random();
      List<Participant> assignedList = null;
      bool validAssignment = false;
      int attempts = 0;
      int maxAttempts = 100;
  
      while (!validAssignment && attempts < maxAttempts)
      {
          assignedList = participants.OrderBy(x => random.Next()).ToList();
          validAssignment = true;
          for (int i = 0; i < participants.Count; i++)
          {
              if (assignedList[i].UserId == participants[i].UserId)
              {
                  validAssignment = false;
                  break;
              }
          }
          attempts++;
      }
  
      if (!validAssignment)
      {
          throw new Exception("Failed to generate a valid Secret Santa assignment without self-assignment.");
      }
  
      for (int i = 0; i < participants.Count; i++)
      {
          participants[i].AssignedUserId = assignedList[i].UserId;
      }
  }
  ```

### Data Access Layer

- **Entity Framework Core & ApplicationDbContext:**  
  The `ApplicationDbContext` class (derived from `IdentityDbContext`) manages the database connection and contains DbSet definitions for entities such as `Game` and `Participant`. EF Core migrations are used to synchronize the database schema with the model.

- **Data Models:**
  - **ApplicationUser:** Extends the built-in Identity user with additional properties like `FullName`.
  - **Game:** Represents a Secret Santa game, including the game name, owner, and a collection of participants.
  - **Participant:** Represents the association between a user and a game, storing assigned user IDs and gift preferences.
  - **View Models:** Such as `RegisterViewModel`, `LoginViewModel`, and `CreateGameViewModel` are used for data transfer between views and controllers.

## Usage Scenario

1. **Registration and Login:**  
   Users register with their email, password, and full name. After registration, they are automatically logged in and redirected to the "My Games" page.

2. **Game Creation:**  
   Logged-in users create a new game by providing a game name and entering email addresses of participants (who must already have accounts). The application then performs a random assignment to ensure no one is assigned to themselves.

3. **Game Management:**  
   Users view a list of games they participate in and can click on a game to see detailed information, including the full name of the assigned participant and their gift preferences. Users can also update their own gift preferences.

4. **Logout:**  
   Users can log out using the option provided in the navigation bar.

## License

This project is licensed under the [MIT License](LICENSE).

## Author

Bartłomiej Dziura
