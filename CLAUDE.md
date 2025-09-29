# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a JavaScript-based Super Mario game (infinite Mario) that has been modified for learning GitOps practices in a DevSecOps pipeline. It's an HTML5 Canvas-based game originally ported from Notch's Java version.

## Architecture

### High-Level Structure
- **`webapp/`** - Main application containing the Mario game
  - **`Enjine/`** - Custom JavaScript game engine with modular components
  - **`code/`** - Game-specific logic and entities
  - **`images/`** - Sprite assets and game graphics
  - **`sounds/`** - Audio files (background music works best in Firefox)
  - **`index.html`** - Main entry point that loads all game modules

### Key Components

**Game Engine (Enjine)**:
- Modular architecture with separate files for each component
- Core systems: `core.js`, `gameCanvas.js`, `keyboardInput.js`, `resources.js`
- Rendering: `drawable.js`, `drawableManager.js`, `sprite.js`, `animatedSprite.js`
- Game state: `state.js`, `gameTimer.js`, `camera.js`

**Game Logic (code/)**:
- Entity system: `character.js`, `enemy.js`, `bulletBill.js`, `shell.js`, etc.
- Level generation: `level.js`, `levelGenerator.js`, `levelRenderer.js`
- Effects: `fireball.js`, `sparkle.js`, `coinAnim.js`, `particle.js`
- Background: `backgroundGenerator.js`, `backgroundRenderer.js`

### DevSecOps Integration
- **`.github/workflows/gitops.yaml`** - GitOps workflow configuration
- **`sonar-project.properties`** - SonarQube static analysis configuration
- Repository is configured for GitOps practices with automated CI/CD

## Development Commands

### Running the Application
```bash
# Serve the webapp directory with any HTTP server
# Examples:
python -m http.server 8000 --directory webapp
# or
npx serve webapp
# or
cd webapp && php -S localhost:8000
```

Then open `http://localhost:8000` in a web browser.

### Testing
No automated test framework is configured. Testing is manual through browser interaction.

### Static Analysis
```bash
# SonarQube analysis (requires SonarQube server setup)
sonar-scanner
```

## File Organization

- All game assets are self-contained in `webapp/`
- JavaScript modules are loaded in dependency order via `index.html`
- No build process required - runs directly in browser
- Game controls: Arrow keys for movement, 'S' key for jumping/actions

## Development Notes

- This is a pure JavaScript/HTML5 project with no Node.js dependencies
- Audio may have browser compatibility issues (Firefox recommended)
- Game uses Canvas API extensively for rendering
- No modern JavaScript framework - uses vanilla JS and jQuery 1.4.2