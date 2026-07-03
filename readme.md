Improv Show Generator - Project Plan
Project Overview
An application that generates improv show schedules by intelligently assigning players to games while optimizing for fairness and engagement.

Core Objectives
Even Distribution: Each player should receive approximately equal number of games
Minimize Downtime: No player should be offstage for more than X consecutive games (configurable threshold)
Valid Assignment: Each game must have the correct number of players assigned
Flexibility: Allow manual adjustments while maintaining validation
Data Models
Player
firstName: string
lastName: string
experienceLevel: enum/string (e.g., "Beginner", "Intermediate", "Advanced")
bio: text
imageUrl: string (optional)
Game
name: string (e.g., "Alphabet", "Three Headed Expert")
requiredPlayers: integer (number of players needed)
description: text
difficultyLevel: enum/string (e.g., "Easy", "Medium", "Hard")
isGroupGame: boolean (if true, requires all players in the show)
order: integer (optional, specifies fixed position in show; typically used for opening/closing games)
duration: integer (optional, duration in minutes; for future show length calculation)
Show
players: array of Player objects
availableGames: array of Game objects
selectedGames: array of GameAssignment objects
maxConsecutiveGamesOff: integer (configurable threshold)
GameAssignment
order: integer (sortable position in show)
game: Game object reference
assignedPlayers: array of Player objects
playerAssignmentMap: object/map (playerId: boolean) for quick lookup
Show Generation Algorithm
Phase 1: Game Selection & Fixed-Order Handling
Select games from available game library
Lock fixed-order games: Games with specified order property are locked to those positions
Games with order: 1, 2, 3... are fixed to the beginning
Games with order: 999 (or high values) are fixed to the end
Games without an order property are flexible and reordered as needed
Evaluate overflow games: If main games don't achieve fair distribution, analyze overflow games to determine which (if any) improve fairness
Calculate variance with current game set
Test each overflow game to see variance impact
Include only overflow games that reduce variance (improve fairness)
Example: If 2 players need 1 more game and overflow has a 2-player game, include just that one; don't add a 3-player game that would make distribution worse
Calculate total show length based on locked + flexible + selected overflow games
Phase 2: Player Assignment Algorithm
Input: List of selected games (including any overflow games chosen), list of players

Constraints:

Games with fixed order property must appear at specified positions
Each game must have exactly the required number of players
Minimize variance in total games per player
Minimize consecutive games a player is offstage
Algorithm Approach:

Initialize: Create assignment matrix (games × players)
Group Games First: Assign all players to any group games
Calculate Initial Distribution:
Target games per player = (total game slots / number of players)
Note: Perfect equality may not be achievable due to math constraints
Aim to minimize variance (difference between max and min games per player)
Iterative Assignment:
For each remaining game:
Calculate score for each player based on:
Current game count (prefer players with fewer games)
Games since last appearance (prefer players who've been off longer)
Avoid same player combinations when possible
Select top N players with highest scores
Optimize Order: Reorder games to minimize consecutive downtime
Use constraint satisfaction or greedy ordering
Check each player's gaps between appearances
Phase 3: Validation & Manual Adjustment
Calculate metrics for each assignment
Flag games that need manual review
Provide suggestions for improvements
Output Format (Spreadsheet-like)
Column Structure
Order	Game Name	Required Players	Player 1	Player 2	...	Player N	Player Count	Status
1	Alphabet	4	1	0	...	1	4	MATCH
2	Expert	3	0	1	...	1	3	MATCH
3	Scene	2	1	1	...	0	2	MATCH
...	...	...	...	...	...	...	...	...
TOTAL			3	2	...	2		
Columns Explained:
Order: Numeric value for sorting (allows manual reordering)
Game Name: Name of the improv game
Required Players: Expected number of players for this game
Player Columns: One column per player (1 = assigned, 0 = not assigned)
Player Count: SUM of all player columns (calculated) - horizontal sum for each game
Status: "MATCH" if Player Count = Required Players, else "FIX"
Summary Row (Bottom):
TOTAL Row: At the bottom of the spreadsheet, sum each player column vertically
Shows total number of games assigned to each player
Purpose:
Quick visual check of distribution fairness
Identifies players who are over/under-assigned
Note: Perfect equality isn't always possible due to math (e.g., 7 games with 3-player games and 5 players)
Manual adjustment may be needed to balance as much as possible
Additional Metrics Display:
Per Player Summary (shown in TOTAL row at bottom):
Total games assigned (vertical sum of each player column)
Visual indication of distribution fairness
Highlights over/under-assigned players
Extended Per Player Metrics (optional detailed view):
Max consecutive games offstage
Game distribution (beginning/middle/end of show)
Show Summary:
Total games
Average games per player
Distribution variance
Target games per player (ideal if perfectly divisible)
User Interface Requirements
Workflow
Load Data: GUI fetches players and games from local data store (JSON files)
Select: User selects which games and players for the show
Configure: User sets generation options (max downtime, strategy, etc.)
Generate: GUI sends request to generation API endpoint
Review: Display generated assignments in spreadsheet format
Edit: Allow manual adjustments
Save: Store final show to data store
Input Section
Player Management:

Load players from data store
Add/remove players
Import from file (CSV/JSON)
Edit player details
Select which players are in this show
Game Library:

Browse available games from data store
Filter by difficulty, number of players
Add custom games
Select which games for this show
Show Configuration:

Show title and date
Set max consecutive games off threshold
Choose optimization strategy
Set other generation options
Generation Section
"Generate Show" button
Sends API request with selected players, games, and config
Loading indicator during generation
Display generation metrics and warnings
Option to regenerate with different seed/settings
Output Section
Interactive grid/table showing assignments (spreadsheet format)
Ability to:
Manually toggle player assignments
Drag to reorder games
Real-time validation updates
Highlight issues (FIX status)
View player totals and metrics
"Regenerate" vs "Manual Edit" modes
Export Options
Save show to data store (JSON file)
Export to CSV/Excel
Export to printable program format
Program Generation (Future Feature)
Program Document Should Include:
Show Title & Date
Player Bios Section:
Photo (if available)
Name
Bio
Experience level
Show Order Section:
Game name
Brief description
Players performing in each game
Formatting:
Professional layout
Customizable templates
PDF export
Architecture
Service Layer Design (Hexagonal Architecture)
Core Principle: Build reusable service classes with core business logic, then add adapters (HTTP API, CLI, GUI) around them. Services receive/return plain data objects - no framework dependencies.

Benefits:

Pure business logic separate from transport layer (HTTP, CLI, GUI)
Easy to unit test without web framework complexity
Reusable as a library in other projects
Multiple interfaces (API, CLI, GUI) can all use same services
Fast MVP without HTTP framework overhead
Architecture Flow (Service Layer First):

Core Services (business logic)
  ├── ShowGeneratorService
  ├── ShowValidatorService
  └── WorksheetService
       ↓
Can be called by multiple adapters:
  ├── Direct calls (testing, CLI)
  ├── HTTP API adapter (Phase 2)
  ├── CLI adapter (Phase 1 optional)
  └── GUI adapter (Phase 3)

MVP Workflow:
User/CLI → Call Service (e.g., showGeneratorService.generate())
        → Service receives request object
        → Service returns response object
        → Service has no knowledge of HTTP/GUI/CLI
        → In Phase 2: HTTP endpoint becomes thin wrapper around service
Workflow: Users can refine shows iteratively by:

Calling showGeneratorService.generate(request) with players and games
Reviewing the output assignments and metrics
Adjusting the game list/order in the request object
Calling service again with refined request
Repeat until satisfied
Store final show via data access layer
Repeating until desired result achieved
API Endpoint Design
Endpoint: POST /api/generate-show

Request Body:

{
  "players": [
    {
      "id": "player-1",
      "firstName": "Jane",
      "lastName": "Doe",
      "experienceLevel": "Advanced"
    }
  ],
  "games": [
    {
      "id": "game-1",
      "name": "Alphabet",
      "requiredPlayers": 2,
      "difficultyLevel": "Easy",
      "isGroupGame": false,
      "order": null,
      "duration": 5
    },
    {
      "id": "game-2",
      "name": "Group Finale",
      "requiredPlayers": 5,
      "difficultyLevel": "Easy",
      "isGroupGame": true,
      "order": 999,
      "duration": 10
    }
  ],
  "overflowGames": [
    {
      "id": "game-3",
      "name": "Improv Backlog",
      "requiredPlayers": 2,
      "difficultyLevel": "Medium",
      "isGroupGame": false,
      "duration": 5
    }
  ],
  "config": {
    "maxConsecutiveGamesOff": 2,
    "optimizationStrategy": "balanced",
    "allowPartialGames": false,
    "seed": 12345
  }
}
Configuration Options:

maxConsecutiveGamesOff: integer (default: 2)
optimizationStrategy: "speed" | "balanced" | "optimal" (algorithm choice)
allowPartialGames: boolean (allow games with fewer than required players)
seed: integer (optional, for reproducible randomization; if omitted, generates randomly each call)
prioritizeExperience: boolean (assign less experienced players to easier games)
balanceStrategy: "equal-games" | "minimize-downtime" | "balanced"
Response Body:

{
  "success": true,
  "showAssignments": [
    {
      "order": 1,
      "gameId": "game-1",
      "gameName": "Alphabet",
      "requiredPlayers": 2,
      "assignedPlayerIds": ["player-1", "player-3"],
      "isValid": true
    }
  ],
  "metrics": {
    "totalGames": 10,
    "playerGameCounts": {
      "player-1": 5,
      "player-2": 4,
      "player-3": 5
    },
    "maxConsecutiveOff": {
      "player-1": 2,
      "player-2": 1,
      "player-3": 2
    },
    "averageGamesPerPlayer": 4.67,
    "distributionVariance": 0.22
  },
  "warnings": [
    "Player 'John Smith' has 3 consecutive games off (exceeds threshold of 2)"
  ]
}
Error Response:

{
  "success": false,
  "error": "INSUFFICIENT_PLAYERS",
  "message": "Cannot assign 5 players to a game requiring 6",
  "details": {
    "gameId": "game-3",
    "required": 6,
    "available": 5
  }
}
Show Validation Endpoint
Endpoint: POST /api/validate-show

Purpose: Analyze a proposed show configuration before generation to provide feedback and suggestions for improvement.

Request Body (same structure as /api/generate-show):

{
  "players": [...],
  "games": [...],
  "overflowGames": [...],
  "config": {...}
}
Response Body:

{
  "success": true,
  "isBalanced": false,
  "analysis": {
    "totalGameSlots": 12,
    "playerCount": 5,
    "slotsPerPlayer": 2.4,
    "targetSlotsPerPlayer": 2,
    "playersWithLessGames": ["player-1", "player-4"],
    "playersWithMoreGames": ["player-2", "player-3"],
    "gamesPerPlayerDistribution": {
      "player-1": 2,
      "player-2": 3,
      "player-3": 3,
      "player-4": 2,
      "player-5": 2
    }
  },
  "suggestions": [
    {
      "type": "IMBALANCED_DISTRIBUTION",
      "severity": "warning",
      "message": "2 players will have fewer games than others",
      "detail": "Players player-1 and player-4 will have 2 games while player-2 and player-3 will have 3 games. Standard deviation: 0.49",
      "recommendation": "Add a 2-player game to the game list to give those players one more slot. This will result in all players having equal games."
    },
    {
      "type": "OVERFLOW_CAN_FIX",
      "severity": "info",
      "message": "Overflow games can fix this imbalance",
      "detail": "Overflow game 'Backlog Game 1' (2-player) will perfectly balance all players. Adding it will give all players exactly 3 games.",
      "recommendation": "When you call /api/generate-show, the overflow game will be automatically included to achieve perfect balance."
    },
    {
      "type": "CONSTRAINTS_MET",
      "severity": "info",
      "message": "All hard constraints are satisfied",
      "detail": "Enough total player slots for the player count. No games exceed available players."
    }
  ],
  "warnings": [],
  "overflow_impact_analysis": [
    {
      "gameId": "overflow-game-1",
      "gameName": "Quick Pair Game",
      "requiredPlayers": 2,
      "currentVariance": 0.49,
      "varianceIfAdded": 0,
      "shouldInclude": true,
      "reason": "Eliminates imbalance. All players will have exactly 3 games."
    },
    {
      "gameId": "overflow-game-2",
      "gameName": "Group Activity",
      "requiredPlayers": 3,
      "currentVariance": 0.49,
      "varianceIfAdded": 0.71,
      "shouldInclude": false,
      "reason": "Would make distribution worse. Not recommended."
    }
  ],
  "gamesForGeneration": [
    "game-1", "game-2", "game-3", "overflow-game-1"
  ]
}
Validation Checks:

Hard Constraints:

Sufficient total player slots for all players
No game requires more players than available
Fixed-order games don't conflict (enough players for fixed games)
Soft Constraints:

Distribution fairness (standard deviation of games per player)
All players have at least 1 game
Downtime threshold can be met
Overflow Game Suggestions:

Analyze which overflow games (if any) would improve fairness
Calculate the impact of adding each overflow game on distribution variance
Recommend only the minimal set of overflow games needed to achieve better fairness
Example: If 2 players need 1 more game, suggest the single overflow game that requires 2 players, not additional games
Create Blank Worksheet Endpoint
Endpoint: POST /api/create-worksheet

Purpose: Generate a blank spreadsheet-like worksheet with games and players pre-populated, but no player assignments. Users can print this and manually fill in the assignments.

Request Body (same structure as /api/generate-show):

{
  "players": [...],
  "games": [...],
  "config": {...}
}
Response Body:

{
  "success": true,
  "worksheet": {
    "columns": ["Order", "Game Name", "Required Players", "Player 1", "Player 2", "Player 3", "Player 4", "Player 5", "Player Count", "Status"],
    "rows": [
      {
        "order": 1,
        "gameName": "Alphabet",
        "requiredPlayers": 2,
        "assignments": [0, 0, 0, 0, 0],
        "playerCount": 0,
        "status": "blank"
      },
      {
        "order": 2,
        "gameName": "Three Headed Expert",
        "requiredPlayers": 3,
        "assignments": [0, 0, 0, 0, 0],
        "playerCount": 0,
        "status": "blank"
      }
    ],
    "totalsRow": {
      "label": "TOTAL",
      "totals": [0, 0, 0, 0, 0],
      "playerNames": ["Jane Doe", "John Smith", "Alice Johnson", "Bob Williams", "Carol Davis"]
    }
  },
  "metrics": {
    "totalGames": 2,
    "playerCount": 5,
    "targetSlotsPerPlayer": 2,
    "note": "Blank worksheet - user should aim for approximately equal assignments per player"
  }
}
Use Case:

User wants to generate a printable blank form
User manually assigns players to games by marking 1 or 0
Can be used as a worksheet for a meeting where decisions are made collaboratively
Output can be exported to CSV/Excel for printing
Client-Side vs. Server-Side Generation
Option A: Server-Side API (Recommended for complex optimization)

Node.js/Python/Java service
Can use OR-Tools or other heavy libraries
Better for complex constraint solving
Easier to update algorithm without client updates
Option B: Client-Side Only (Simpler deployment)

Pure JavaScript/WASM in browser
No server required
Works offline
Faster (no network round-trip)
Good for greedy algorithm approach
Hybrid Approach (Best of both):

Client-side greedy algorithm for instant results
Optional server-side optimization for better results
Fallback if server unavailable
Technical Considerations
Technology Stack Options
MVP (Service Layer - Phase 1): ✅

Language: C# (.NET)
Testing: xUnit or NUnit
JSON Serialization: System.Text.Json or Newtonsoft.Json
No HTTP framework required - pure business logic classes
Phase 2+ (Adding HTTP API):

Web Framework: ASP.NET Core (natural fit with C#)
Desktop App: WPF or Windows Forms
Mobile: MAUI (if adding mobile later)
Alternative considerations (Phase 2+):

Could also expose services via gRPC or other transport
JavaScript/TypeScript version possible if cross-platform needed
Python version possible for data science workflows
Data Storage
Recommended MVP: JSON Files (File-based)
Pros:

Simplest to implement - no database setup
Human-readable and easy to debug
Version control friendly (Git)
Easy to backup and share
No dependencies or setup required
Perfect for single-user desktop/web app
Structure (MVP - project directory):

project-root/
  ├── data/
  │   ├── players.json          # Array of all player objects
  │   ├── games-library.json    # Array of all available games
  │   └── shows/
  │       ├── show-2026-06-29.json    # Individual show files
  │       ├── show-2026-07-15.json
  │       └── ...
  ├── src/
  │   └── api/
  │       └── generate-show.js   # API implementation
  └── package.json
File Schemas:

players.json:

[
  {
    "id": "uuid-1",
    "firstName": "Jane",
    "lastName": "Doe",
    "experienceLevel": "Advanced",
    "bio": "Jane has been performing improv...",
    "imageUrl": "https://example.com/jane.jpg"
  }
]
games-library.json:

[
  {
    "id": "uuid-1",
    "name": "Alphabet",
    "requiredPlayers": 2,
    "description": "Two players perform a scene...",
    "difficultyLevel": "Easy",
    "isGroupGame": false
  }
]
show-YYYY-MM-DD.json:

{
  "id": "uuid-1",
  "title": "Summer Improv Night",
  "date": "2026-06-29",
  "maxConsecutiveGamesOff": 2,
  "players": ["player-id-1", "player-id-2"],
  "gameAssignments": [
    {
      "order": 1,
      "gameId": "game-id-1",
      "assignedPlayerIds": ["player-id-1", "player-id-3"]
    }
  ]
}
Migration Path to Database
Phase 1 (MVP): JSON files

Read/write JSON files using file system API
Keep data structure identical to what database will use
Use IDs/references instead of nested objects
Phase 2 (Database):

Option A - SQLite:

Perfect for desktop apps
Single file database
No server required
Easy to bundle with Electron
npm: better-sqlite3 or sql.js
Option B - PostgreSQL/MySQL:

For multi-user or cloud deployment
More setup but scales better
Cloud options: Supabase, PlanetScale, Neon
Option C - IndexedDB (for web apps):

Browser-based database
No backend needed
Works offline
Libraries: Dexie.js, localForage
Migration Strategy:

Data Access Layer (DAL): Create abstraction from day one

// Example interface
interface DataStore {
  getPlayers(): Promise<Player[]>
  savePlayer(player: Player): Promise<void>
  getShows(): Promise<Show[]>
  // ... etc
}

// Implementations:
class JsonFileStore implements DataStore { }
class SQLiteStore implements DataStore { }
class PostgresStore implements DataStore { }
JSON → Database: Write migration script

Read all JSON files
Insert into database tables
Verify data integrity
Keep JSON as backup during transition
Schema Parity: Design JSON structure to match database tables

Use IDs not nested objects (easier to convert)
Normalize data (separate players from shows)
Think in terms of tables/collections
Storage by Platform
Web App (Browser):

MVP: LocalStorage + JSON serialization
Better: IndexedDB (Dexie.js)
Future: Backend API + PostgreSQL
Desktop App (Electron):

MVP: JSON files in user's Documents folder
Better: SQLite database
Future: Optional cloud sync
Mobile App:

MVP: AsyncStorage + JSON
Better: SQLite (expo-sqlite, react-native-sqlite-storage)
Future: Cloud sync
Recommendation for MVP
Start with JSON files in project directory because:

Zero setup - works immediately
Easy to understand and debug
Can inspect/edit files manually if needed
Version control friendly for development
Clean migration path to database when needed
Project directory keeps everything self-contained for MVP
Easy to share/test configurations
When to migrate to database:

Need multi-user access
Data size becomes significant (> 1000 players/shows)
Need complex queries or relationships
Want real-time sync across devices
Need transaction support for data integrity
Scheduling Algorithm Libraries
The player assignment problem is a constraint satisfaction problem (CSP) with optimization goals. Several open-source libraries can help:

Recommended: Google OR-Tools (CP-SAT Solver)
Best fit for this problem
Languages: Python, Java, C++, .NET, JavaScript (via WASM)
Use case: Constraint programming and optimization
Pros:
Excellent for scheduling/assignment problems
Can handle hard constraints (exact player count) and soft constraints (fairness, minimize downtime)
Fast and well-documented
Free and open source (Apache 2.0)
Example constraints it can handle:
Each game has exactly N players
Minimize variance in games per player
Minimize maximum consecutive games off
No player in consecutive games (if needed)
Link: https://developers.google.com/optimization
Alternative Libraries
For Python:

python-constraint: Simpler CSP library, easier learning curve
PuLP: Linear programming, good for optimization
DEAP: Genetic algorithms, useful for complex optimization
For JavaScript/TypeScript:

constraint-solver (npm): Basic CSP for JavaScript
jsil: Logic programming in JavaScript
OR-Tools via WASM: Google OR-Tools compiled to WebAssembly
genetic-js: Genetic algorithm approach
For Java/JVM:

Choco-solver: Powerful constraint programming
OptaPlanner: Specifically designed for planning/scheduling problems
JaCoP: Java constraint programming
Algorithm Approach Options
Custom Greedy Algorithm (MVP approach):

Simplest to implement
Good enough for typical shows
Easy to understand and debug
May not find optimal solution
Constraint Satisfaction (OR-Tools):

More sophisticated
Guarantees valid solutions
Can optimize multiple objectives
Requires learning curve
Hybrid Approach (Recommended):

Use greedy algorithm for initial assignment
Use constraint solver for optimization/reordering
Fallback to manual adjustment
Best balance of simplicity and power
Algorithm Performance
Should handle typical show sizes (6-15 players, 8-20 games) instantly
May need optimization for larger shows
Consider caching and memoization
Constraint solvers typically find solutions in < 1 second for this problem size
Implementation Phases
Phase 1: MVP (Minimum Viable Product - Service Layer)
Deliverable: Reusable C# service classes with core business logic, fully unit tested

Technology Stack:

Language: C# (.NET 6+)
Testing Framework: xUnit
JSON: System.Text.Json
No HTTP framework (pure business logic)
Project Structure:

ImprovShowGenerator/
  ├── src/
  │   └── ImprovShowGenerator/
  │       ├── Services/
  │       │   ├── ShowGeneratorService.cs
  │       │   ├── ShowValidatorService.cs
  │       │   └── WorksheetService.cs
  │       ├── Models/
  │       │   ├── Player.cs
  │       │   ├── Game.cs
  │       │   ├── Show.cs
  │       │   ├── GenerateShowRequest.cs
  │       │   ├── GenerateShowResponse.cs
  │       │   └── ... (other DTOs)
  │       ├── Utils/
  │       │   ├── ShowGenerationAlgorithm.cs
  │       │   └── FairnessCalculator.cs
  │       └── Data/
  │           ├── IDataStore.cs (interface)
  │           └── JsonFileStore.cs (implementation)
  ├── tests/
  │   └── ImprovShowGenerator.Tests/
  │       ├── Services/
  │       │   ├── ShowGeneratorServiceTests.cs
  │       │   ├── ShowValidatorServiceTests.cs
  │       │   └── WorksheetServiceTests.cs
  │       ├── Utils/
  │       │   └── ShowGenerationAlgorithmTests.cs
  │       └── TestFixtures/
  │           └── SampleData.cs
  ├── data/
  │   ├── players.json
  │   ├── games-library.json
  │   └── shows/
  └── ImprovShowGenerator.csproj
Core Service Implementation:

[ ] Define data structures as C# classes (Player, Game, Show, Request, Response models - POCOs)
[ ] Design service interfaces matching the proposed endpoint contracts
[ ] Implement ShowGeneratorService class
[ ] generate(request) method - stateless show generation
[ ] Player assignment algorithm (greedy approach)
[ ] Handle fixed-order games
[ ] Implement overflow games logic
[ ] Support optional seed parameter for reproducible results
[ ] Implement ShowValidatorService class
[ ] validate(request) method - pre-generation analysis
[ ] Hard constraint validation
[ ] Fairness metrics calculation
[ ] Overflow game impact analysis
[ ] Implement WorksheetService class
[ ] createWorksheet(request) method - blank spreadsheet generation
Data Management:

[ ] Create data access layer abstraction (generic interface for storage implementations)
[ ] Implement JSON file storage adapter in project directory (players, games, shows)
[ ] File I/O utilities for loading/saving data
[ ] Sample/fixture data files for testing
Comprehensive Testing:

[ ] Unit tests for ShowGeneratorService (distribution fairness, algorithm correctness)
[ ] Unit tests for ShowValidatorService (constraint checking, metrics)
[ ] Unit tests for WorksheetService (blank worksheet generation)
[ ] Unit tests for overflow games logic
[ ] Unit tests for fixed-order games handling
[ ] Unit tests for edge cases and error handling
[ ] Test fixtures and sample data scenarios
[ ] Test coverage target: > 80% of service logic
Documentation:

[ ] Service interfaces documented with XML documentation comments (///)
[ ] Request/response schemas documented (C# types and serialization)
[ ] Usage examples showing how to call services (C# code examples)
[ ] Architecture documentation (Hexagonal Architecture pattern explanation)
[ ] Optional: Generate HTML docs with Sandcastle or DocFX
Phase 2: HTTP API & Algorithm Enhancement
Deliverable: REST API wrapping services + improved algorithm options

HTTP API Implementation:

[ ] Setup web framework (Express, Fastify, or similar)
[ ] Implement /api/generate-show endpoint (wraps ShowGeneratorService.generate())
[ ] Implement /api/validate-show endpoint (wraps ShowValidatorService.validate())
[ ] Implement /api/create-worksheet endpoint (wraps WorksheetService.createWorksheet())
[ ] HTTP error handling and validation
[ ] API documentation (Swagger/OpenAPI)
Enhanced Algorithm:

[ ] Evaluate constraint solver integration (OR-Tools or similar)
[ ] Implement downtime minimization
[ ] Add reordering optimization
[ ] Multiple algorithm strategies (user can choose)
[ ] Performance metrics and benchmarking
[ ] Algorithm comparison testing
Phase 3: GUI & User Interface
[ ] Build GUI to call services (web or desktop)
[ ] Game library management interface
[ ] Player database interface
[ ] Interactive show editor (spreadsheet-like grid)
[ ] Validation feedback UI (show suggestions before generation)
[ ] Manual assignment/editing capability (toggle player assignments, drag to reorder)
[ ] Real-time validation updates with MATCH/FIX indicators
[ ] Player totals and fairness metrics display
[ ] "Regenerate" vs "Manual Edit" modes
Phase 4: Export & Program Generation
[ ] CSV/Excel export functionality
[ ] Save/load show configurations
[ ] Program template system
[ ] PDF generation with player bios
[ ] Customizable formatting
Phase 5: Advanced Features
[ ] Player preferences and conflicts (mark avoided/preferred games)
[ ] Experience level factoring into assignments
[ ] History and versioning (track changes to shows)
[ ] Templates for common show formats
[ ] Analytics (player participation over multiple shows)
[ ] Custom constraint support (e.g., max appearances, skill-based grouping)
[ ] Multi-show analysis and reporting
Open Questions & Decisions Needed
MVP Scope: ✅ Service Layer (no HTTP API, no GUI initially) - focus on reusable services with comprehensive unit tests
MVP Language: ✅ C# (.NET) - service layer implementation with xUnit tests
Data Storage: ✅ Start with JSON files, migrate to database later
Data Location: ✅ Project directory (for MVP)
Architecture: ✅ Service Layer Pattern (Hexagonal Architecture)
Fixed-Order Games: ✅ Games with order: 999 locked to end position (e.g., group finale)
Overflow Games: ✅ Automatically included only when they improve fairness (reduce distribution variance)
Randomization: ✅ Each call generates randomly. Users refine by calling service with adjusted games list from previous output
Optional seed parameter for reproducible results during testing
Show Duration: ⏳ Optional duration property on games (for future show length calculation, not MVP priority)
Conflict/Preference: Deferred (not important for MVP) - add in Phase 3+
Still to Decide: 11. Phase 2 HTTP Framework: ASP.NET Core (natural fit with C#) or alternative? 12. Algorithm Complexity: Custom greedy vs. constraint solver (OR-Tools)? - MVP: Start with greedy, add solver later? - Or use OR-Tools from the start for better results? 13. Downtime Threshold: Default value? (Suggest: 2) 14. Experience Level: Should it factor into game assignments? 15. Game Library: Pre-populated or user-created?

Success Metrics
Show generation time: < 2 seconds for typical show
Assignment fairness: Standard deviation of games per player < 1
Downtime optimization: No player off > threshold (configurable, default 2-3 games)
User satisfaction: Reduces manual planning time by 80%+
Notes
Notes
The spreadsheet format is familiar to current users, maintain compatibility
Manual override is essential - algorithm provides starting point
Validation is key - easy to see when assignments need adjustment
Consider both novice and expert user workflows
JSON files provide simplest starting point with clear database migration path
Data access layer abstraction from day one enables smooth future database migration
Service layer design enables testing without web framework complexity
Services are pure functions: same inputs = same outputs (with same seed)
Services can be called by HTTP API (Phase 2), CLI (optional), or GUI (Phase 3)
Each adapter (HTTP, CLI, GUI) is just a thin wrapper around services
C# MVP provides strong typing and excellent testing framework (xUnit)
ASP.NET Core Phase 2 will integrate seamlessly with C# services
.NET provides excellent cross-platform support (Windows, Linux, macOS)
MVP Workflow (Service Layer Approach - C#)
Development Workflow:

Define request/response models as C# classes (POCOs)
Implement ShowGeneratorService, ShowValidatorService, WorksheetService classes
Write comprehensive xUnit tests for each service
Call services directly in tests (no HTTP framework needed)
Validate algorithm correctness through unit tests
Update tests as requirements evolve
Usage Pattern in Tests and Phase 2+:

Client/Code calls showGeneratorService.Generate(request) with players, games, config
Service returns GenerateShowResponse with assignments and metrics
Client reviews the assignments and fairness metrics
If unsatisfied, client adjusts game list/order and calls service again
Repeat until desired result achieved
Save final show to data store via data access layer
Later in Phase 3: GUI wraps these service calls
Key Design Points:

Each service call is independent and stateless (all data in request object)
Randomization makes each call produce different results (unless seed is provided)
Service layer has no HTTP dependencies - pure C# business logic
Optional seed parameter enables reproducible results for testing/refinement
Strong typing with C# classes prevents runtime errors
Easy to test - simple unit tests without mocking HTTP layers
Phase 2 HTTP endpoints will be thin wrappers around these services