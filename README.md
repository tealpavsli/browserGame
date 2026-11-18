# Heart Paddle — a browser arcade game

A responsive arcade game built with **Blazor WebAssembly (.NET 10, C#)**. You control a paddle and bounce a heart into the goal at the top of the field. Every goal speeds the game up and shrinks the paddle, so a run always ends in a loss — the point is to beat your own record. It runs entirely in the browser on desktop and mobile, and player records are stored online.

**Play it:** https://tealpavsli.github.io/browserGame/

> The in-game interface and messages are in Russian.

## Features

**Gameplay**
- Paddle control with the mouse or by touch (Pointer Events — one code path for both).
- 5 lives. Losing one resets the ball onto the paddle; losing all of them ends the run.
- Ball speed grows 5% after each goal and drops 5% after each lost life. The base speed scales with the field size, so difficulty feels the same on a phone and on a monitor.
- The paddle shrinks 1% after every goal and keeps its size when a life is lost.
- Hitting the paddle near its edge deflects the ball sideways, so the player can aim.
- Progress (score, lives, record) survives screen rotation and window resizing.

**Customization**
- 5 ball skins: red, blue and green hearts, a black square and a purple diamond.
- 5 field colors. Both can only be changed while the ball is waiting on the paddle.

**Feedback**
- Random motivational messages after goals and consoling ones after lost lives, never repeated back to back.
- A celebration animation on every 5th goal, cycling through: confetti, applause, a golden trophy, a thumbs-up and a podium.

**Online records**
- Enter a name to load your saved record; a new name creates a new entry. No passwords — the game is made for a small circle of players.
- Records are kept in **Firebase Firestore** and are available from any device.

## Tech stack

| Area | Technology |
| --- | --- |
| Game logic | C# / .NET 10, Blazor WebAssembly |
| Rendering | HTML5 Canvas 2D, JavaScript interop, `requestAnimationFrame` |
| UI overlays | Razor components, CSS animations and SVG |
| Storage | Firebase Firestore (called straight from the browser, no custom backend) |
| Tests | xUnit |
| Hosting / CI | GitHub Pages, GitHub Actions |

## How it works

The project separates the game rules from the drawing:

- `GameEngine/` is plain C# with no UI dependencies: vectors, field, paddle, ball, collision resolution and the game session (score, lives, speed, record). It works in abstract field units, knows nothing about pixels, and is covered by unit tests.
- On every animation frame the browser calls the C# `OnTick` method through JS interop. C# advances the physics and returns the object positions; JavaScript draws them on the canvas and forwards mouse/touch input back to C#.
- Score, lives, messages, the game-over screen and the celebration animations are regular HTML/CSS overlays on top of the canvas.

## Project structure

```
project2/                     Blazor WebAssembly game
├── GameEngine/               physics, collisions, game session (pure C#)
├── Services/                 motivational message provider
├── Pages/GameCanvas.razor    the game page: UI state and JS interop
└── wwwroot/
    ├── js/gameInterop.js     canvas rendering, input, game loop
    ├── js/firebaseInterop.js Firestore read/write of player records
    └── css/app.css           overlays, animations, layout
project2.Tests/               xUnit tests for the collision logic
.github/workflows/            build and deploy to GitHub Pages
```

## Run locally

Requires the [.NET 10 SDK](https://dotnet.microsoft.com/download).

```powershell
cd project2
dotnet run
```

Open the address printed in the console (by default `http://localhost:5199`).

Run the tests from the repository root:

```powershell
dotnet test
```

Local runs write records to a separate `players_test` Firestore collection, so testing never touches real player data.

## Deployment

A GitHub Actions workflow builds the project and publishes it to GitHub Pages on every push to `main`. It also builds the `test` branch and publishes it under `/test-preview/`, which makes it possible to check new features on a phone before merging them into `main`.
