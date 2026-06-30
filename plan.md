# Improv Show Generator - Project Plan

## Project Overview
An application that generates improv show schedules by intelligently assigning players to games while optimizing for fairness and engagement.

## Core Objectives
1. **Even Distribution**: Each player should receive approximately equal number of games
2. **Minimize Downtime**: No player should be offstage for more than X consecutive games (configurable threshold)
3. **Valid Assignment**: Each game must have the correct number of players assigned
4. **Flexibility**: Allow manual adjustments while maintaining validation

## Data Models

### Player
- **firstName**: string
- **lastName**: string
- **experienceLevel**: enum/string (e.g., "Beginner", "Intermediate", "Advanced")
- **bio**: text
- **imageUrl**: string (optional)

### Game
- **name**: string (e.g., "Alphabet", "Three Headed Expert")
- **requiredPlayers**: integer (number of players needed)
- **description**: text
- **difficultyLevel**: enum/string (e.g., "Easy", "Medium", "Hard")
- **isGroupGame**: boolean (if true, requires all players in the show)

### Show
- **players**: array of Player objects
- **availableGames**: array of Game objects
- **selectedGames**: array of GameAssignment objects
- **maxConsecutiveGamesOff**: integer (configurable threshold)

### GameAssignment
- **order**: integer (sortable position in show)
- **game**: Game object reference
- **assignedPlayers**: array of Player objects
- **playerAssignmentMap**: object/map (playerId: boolean) for quick lookup

## Show Generation Algorithm

### Phase 1: Game Selection
1. Select games from available game library
2. Determine if group games should be included
3. Calculate total show length based on time/number of games

### Phase 2: Player Assignment Algorithm
**Input**: List of selected games, list of players

**Constraints**:
- Each game must have exactly the required number of players
- Minimize variance in total games per player
- Minimize consecutive games a player is offstage

**Algorithm Approach**:
1. **Initialize**: Create assignment matrix (games × players)
2. **Group Games First**: Assign all players to any group games
3. **Calculate Initial Distribution**:
   - Target games per player = (total game slots / number of players)
   - Note: Perfect equality may not be achievable due to math constraints
   - Aim to minimize variance (difference between max and min games per player)
4. **Iterative Assignment**:
   - For each remaining game:
     - Calculate score for each player based on:
       - Current game count (prefer players with fewer games)
       - Games since last appearance (prefer players who've been off longer)
       - Avoid same player combinations when possible
     - Select top N players with highest scores
5. **Optimize Order**: Reorder games to minimize consecutive downtime
   - Use constraint satisfaction or greedy ordering
   - Check each player's gaps between appearances

### Phase 3: Validation & Manual Adjustment
- Calculate metrics for each assignment
- Flag games that need manual review
- Provide suggestions for improvements

## Output Format (Spreadsheet-like)

### Column Structure
| Order | Game Name | Required Players | Player 1 | Player 2 | ... | Player N | Player Count | Status |
|-------|-----------|------------------|----------|----------|-----|----------|--------------|--------|
| 1     | Alphabet  | 4                | 1        | 0        | ... | 1        | 4            | MATCH  |
| 2     | Expert    | 3                | 0        | 1        | ... | 1        | 3            | MATCH  |
| 3     | Scene     | 2                | 1        | 1        | ... | 0        | 2            | MATCH  |
| ...   | ...       | ...              | ...      | ...      | ... | ...      | ...          | ...    |
| **TOTAL** | | | **3** | **2** | ... | **2** | | |

### Columns Explained:
1. **Order**: Numeric value for sorting (allows manual reordering)
2. **Game Name**: Name of the improv game
3. **Required Players**: Expected number of players for this game
4. **Player Columns**: One column per player (1 = assigned, 0 = not assigned)
5. **Player Count**: SUM of all player columns (calculated) - horizontal sum for each game
6. **Status**: "MATCH" if Player Count = Required Players, else "FIX"

### Summary Row (Bottom):
- **TOTAL Row**: At the bottom of the spreadsheet, sum each player column vertically
- Shows total number of games assigned to each player
- **Purpose**: 
  - Quick visual check of distribution fairness
  - Identifies players who are over/under-assigned
  - Note: Perfect equality isn't always possible due to math (e.g., 7 games with 3-player games and 5 players)
  - Manual adjustment may be needed to balance as much as possible

### Additional Metrics Display:
- **Per Player Summary** (shown in TOTAL row at bottom):
  - Total games assigned (vertical sum of each player column)
  - Visual indication of distribution fairness
  - Highlights over/under-assigned players
- **Extended Per Player Metrics** (optional detailed view):
  - Max consecutive games offstage
  - Game distribution (beginning/middle/end of show)
- **Show Summary**:
  - Total games
  - Average games per player
  - Distribution variance
  - Target games per player (ideal if perfectly divisible)

## User Interface Requirements

### Workflow
1. **Load Data**: GUI fetches players and games from local data store (JSON files)
2. **Select**: User selects which games and players for the show
3. **Configure**: User sets generation options (max downtime, strategy, etc.)
4. **Generate**: GUI sends request to generation API endpoint
5. **Review**: Display generated assignments in spreadsheet format
6. **Edit**: Allow manual adjustments
7. **Save**: Store final show to data store

### Input Section
1. **Player Management**:
   - Load players from data store
   - Add/remove players
   - Import from file (CSV/JSON)
   - Edit player details
   - Select which players are in this show
   
2. **Game Library**:
   - Browse available games from data store
   - Filter by difficulty, number of players
   - Add custom games
   - Select which games for this show
   
3. **Show Configuration**:
   - Show title and date
   - Set max consecutive games off threshold
   - Choose optimization strategy
   - Set other generation options

### Generation Section
- "Generate Show" button
- Sends API request with selected players, games, and config
- Loading indicator during generation
- Display generation metrics and warnings
- Option to regenerate with different seed/settings
  
### Output Section
- Interactive grid/table showing assignments (spreadsheet format)
- Ability to:
  - Manually toggle player assignments
  - Drag to reorder games
  - Real-time validation updates
  - Highlight issues (FIX status)
  - View player totals and metrics
- "Regenerate" vs "Manual Edit" modes
  
### Export Options
- Save show to data store (JSON file)
- Export to CSV/Excel
- Export to printable program format

## Program Generation (Future Feature)

### Program Document Should Include:
1. **Show Title & Date**
2. **Player Bios Section**:
   - Photo (if available)
   - Name
   - Bio
   - Experience level
3. **Show Order Section**:
   - Game name
   - Brief description
   - Players performing in each game
4. **Formatting**:
   - Professional layout
   - Customizable templates
   - PDF export

## Architecture

### API-First Design (Stateless Generation)

**Core Principle**: The show generation logic is a stateless API endpoint that receives all required data in the request.

**Benefits**:
- Clean separation of concerns (data management vs. generation logic)
- Scalable and cacheable
- Testable in isolation
- Can be deployed independently
- Could be reused by multiple clients (web, mobile, CLI)

**Architecture Flow**:
```
GUI/Client → Fetch Games from Data Store
           → Fetch Players from Data Store
           → User Selects Games & Players
           → Build Request Object
           → POST /api/generate-show
           → API generates schedule (no DB access)
           → Returns generated assignments
           → GUI displays and allows edits
           → Save to Data Store when ready
```

### API Endpoint Design

**Endpoint**: `POST /api/generate-show`

**Request Body**:
```json
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
      "isGroupGame": false
    }
  ],
  "config": {
    "maxConsecutiveGamesOff": 2,
    "optimizationStrategy": "balanced",
    "allowPartialGames": false,
    "seed": 12345
  }
}
```

**Configuration Options**:
- `maxConsecutiveGamesOff`: integer (default: 2)
- `optimizationStrategy`: "speed" | "balanced" | "optimal" (algorithm choice)
- `allowPartialGames`: boolean (allow games with fewer than required players)
- `seed`: integer (for reproducible randomization)
- `prioritizeExperience`: boolean (assign less experienced players to easier games)
- `balanceStrategy`: "equal-games" | "minimize-downtime" | "balanced"

**Response Body**:
```json
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
```

**Error Response**:
```json
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
```

### Client-Side vs. Server-Side Generation

**Option A: Server-Side API** (Recommended for complex optimization)
- Node.js/Python/Java service
- Can use OR-Tools or other heavy libraries
- Better for complex constraint solving
- Easier to update algorithm without client updates

**Option B: Client-Side Only** (Simpler deployment)
- Pure JavaScript/WASM in browser
- No server required
- Works offline
- Faster (no network round-trip)
- Good for greedy algorithm approach

**Hybrid Approach** (Best of both):
- Client-side greedy algorithm for instant results
- Optional server-side optimization for better results
- Fallback if server unavailable

## Technical Considerations

### Technology Stack Options
- **Web App**: React/Vue frontend + Node.js/Python API
- **Desktop App**: Electron (API can run locally or remotely)
- **Mobile**: React Native (future)
- **Pure Web**: Client-side only (generation in browser using JavaScript/WASM)

### Data Storage

#### Recommended MVP: JSON Files (File-based)
**Pros**:
- Simplest to implement - no database setup
- Human-readable and easy to debug
- Version control friendly (Git)
- Easy to backup and share
- No dependencies or setup required
- Perfect for single-user desktop/web app

**Structure**:
```
data/
  ├── players.json          # Array of all player objects
  ├── games-library.json    # Array of all available games
  └── shows/
      ├── show-2026-06-29.json    # Individual show files
      ├── show-2026-07-15.json
      └── ...
```

**File Schemas**:

`players.json`:
```json
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
```

`games-library.json`:
```json
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
```

`show-YYYY-MM-DD.json`:
```json
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
```

#### Migration Path to Database

**Phase 1 (MVP)**: JSON files
- Read/write JSON files using file system API
- Keep data structure identical to what database will use
- Use IDs/references instead of nested objects

**Phase 2 (Database)**: 
- **Option A - SQLite**: 
  - Perfect for desktop apps
  - Single file database
  - No server required
  - Easy to bundle with Electron
  - npm: `better-sqlite3` or `sql.js`
  
- **Option B - PostgreSQL/MySQL**:
  - For multi-user or cloud deployment
  - More setup but scales better
  - Cloud options: Supabase, PlanetScale, Neon
  
- **Option C - IndexedDB** (for web apps):
  - Browser-based database
  - No backend needed
  - Works offline
  - Libraries: Dexie.js, localForage

**Migration Strategy**:
1. **Data Access Layer (DAL)**: Create abstraction from day one
   ```javascript
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
   ```

2. **JSON → Database**: Write migration script
   - Read all JSON files
   - Insert into database tables
   - Verify data integrity
   - Keep JSON as backup during transition

3. **Schema Parity**: Design JSON structure to match database tables
   - Use IDs not nested objects (easier to convert)
   - Normalize data (separate players from shows)
   - Think in terms of tables/collections

#### Storage by Platform

- **Web App (Browser)**: 
  - MVP: LocalStorage + JSON serialization
  - Better: IndexedDB (Dexie.js)
  - Future: Backend API + PostgreSQL
  
- **Desktop App (Electron)**:
  - MVP: JSON files in user's Documents folder
  - Better: SQLite database
  - Future: Optional cloud sync
  
- **Mobile App**:
  - MVP: AsyncStorage + JSON
  - Better: SQLite (expo-sqlite, react-native-sqlite-storage)
  - Future: Cloud sync

#### Recommendation for MVP

**Start with JSON files** because:
1. Zero setup - works immediately
2. Easy to understand and debug
3. Can inspect/edit files manually if needed
4. Version control friendly for development
5. Clean migration path to database when needed
6. Your instinct is correct! 

**When to migrate to database**:
- Need multi-user access
- Data size becomes significant (> 1000 players/shows)
- Need complex queries or relationships
- Want real-time sync across devices
- Need transaction support for data integrity

### Scheduling Algorithm Libraries

The player assignment problem is a **constraint satisfaction problem (CSP)** with optimization goals. Several open-source libraries can help:

#### Recommended: Google OR-Tools (CP-SAT Solver)
- **Best fit for this problem**
- **Languages**: Python, Java, C++, .NET, JavaScript (via WASM)
- **Use case**: Constraint programming and optimization
- **Pros**:
  - Excellent for scheduling/assignment problems
  - Can handle hard constraints (exact player count) and soft constraints (fairness, minimize downtime)
  - Fast and well-documented
  - Free and open source (Apache 2.0)
- **Example constraints it can handle**:
  - Each game has exactly N players
  - Minimize variance in games per player
  - Minimize maximum consecutive games off
  - No player in consecutive games (if needed)
- **Link**: https://developers.google.com/optimization

#### Alternative Libraries

**For Python**:
- **python-constraint**: Simpler CSP library, easier learning curve
- **PuLP**: Linear programming, good for optimization
- **DEAP**: Genetic algorithms, useful for complex optimization

**For JavaScript/TypeScript**:
- **constraint-solver** (npm): Basic CSP for JavaScript
- **jsil**: Logic programming in JavaScript
- **OR-Tools via WASM**: Google OR-Tools compiled to WebAssembly
- **genetic-js**: Genetic algorithm approach

**For Java/JVM**:
- **Choco-solver**: Powerful constraint programming
- **OptaPlanner**: Specifically designed for planning/scheduling problems
- **JaCoP**: Java constraint programming

#### Algorithm Approach Options

1. **Custom Greedy Algorithm** (MVP approach):
   - Simplest to implement
   - Good enough for typical shows
   - Easy to understand and debug
   - May not find optimal solution

2. **Constraint Satisfaction (OR-Tools)**:
   - More sophisticated
   - Guarantees valid solutions
   - Can optimize multiple objectives
   - Requires learning curve

3. **Hybrid Approach** (Recommended):
   - Use greedy algorithm for initial assignment
   - Use constraint solver for optimization/reordering
   - Fallback to manual adjustment
   - Best balance of simplicity and power

### Algorithm Performance
- Should handle typical show sizes (6-15 players, 8-20 games) instantly
- May need optimization for larger shows
- Consider caching and memoization
- Constraint solvers typically find solutions in < 1 second for this problem size

## Implementation Phases

### Phase 1: MVP (Minimum Viable Product)
- [ ] Define data structures (JSON schemas)
- [ ] Design API endpoint contract (request/response)
- [ ] Implement stateless generation API endpoint
- [ ] Implement basic player assignment algorithm (greedy approach)
- [ ] Create data access layer abstraction (for future database migration)
- [ ] Implement JSON file storage (players, games, shows)
- [ ] Build GUI for game/player selection
- [ ] Create spreadsheet-like output format with TOTAL row
- [ ] Manual assignment/editing capability
- [ ] Basic validation (MATCH/FIX, player totals)
- [ ] File I/O for saving/loading shows

### Phase 2: Enhanced Algorithm
- [ ] Evaluate constraint solver integration (OR-Tools or similar)
- [ ] Implement downtime minimization
- [ ] Add reordering optimization
- [ ] Multiple algorithm strategies (user can choose)
- [ ] Performance metrics display
- [ ] Algorithm comparison/benchmarking

### Phase 3: User Interface
- [ ] Game library management
- [ ] Player database
- [ ] Interactive show editor
- [ ] Visual feedback and highlighting

### Phase 4: Export & Program Generation
- [ ] CSV/Excel export
- [ ] Program template system
- [ ] PDF generation with bios
- [ ] Customizable formatting

### Phase 5: Advanced Features
- [ ] Save/load show configurations
- [ ] History and versioning
- [ ] Templates for common show formats
- [ ] Analytics (player participation over multiple shows)
- [ ] Constraint customization (e.g., player preferences, conflicts)

## Open Questions & Decisions Needed

1. **Platform**: Web, desktop, or mobile first?
2. **Data Storage**: ✅ Start with JSON files, migrate to database later
3. **API Architecture**: ✅ Stateless generation endpoint (receives all data in request)
4. **API Deployment**: Client-side (JavaScript/WASM) or server-side (Node.js/Python)?
   - Start client-side for simplicity?
   - Or server-side for better algorithms (OR-Tools)?
5. **Algorithm Complexity**: Custom greedy vs. constraint solver (OR-Tools)?
   - MVP: Start with greedy, add solver later?
   - Or use OR-Tools from the start for better results?
6. **User Experience**: How much automation vs. manual control?
7. **Game Library**: Pre-populated or user-created?
8. **Downtime Threshold**: Default value? (Suggest: 2)
9. **Experience Level**: Should it factor into game assignments?
10. **Player Preferences**: Should players be able to mark preferred/avoided games?
11. **Data Location**: Where should JSON files be stored? (User's Documents folder, app data folder, project folder?)

## Success Metrics

- Show generation time: < 2 seconds for typical show
- Assignment fairness: Standard deviation of games per player < 1
- Downtime optimization: No player off > threshold (configurable, default 2-3 games)
- User satisfaction: Reduces manual planning time by 80%+

## Notes

- The spreadsheet format is familiar to current users, maintain compatibility
- Manual override is essential - algorithm provides starting point
- Validation is key - easy to see when assignments need adjustment
- Consider both novice and expert user workflows
- JSON files provide simplest starting point with clear database migration path
- Data access layer abstraction from day one enables smooth future database migration
- Stateless API design separates generation logic from data management
- GUI orchestrates: fetch data → call API → display results → save if satisfied
- Generation endpoint is pure function: same inputs = same outputs (with same seed)
