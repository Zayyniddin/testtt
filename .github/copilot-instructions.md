# Copilot Instructions for Birthday Cake Visualization

## Project Overview
This is an **interactive 3D birthday celebration experience** built with React Three Fiber. It presents an animated scene with a cake, candles, fireworks, and birthday cards - designed as a gift for a special person.

### Key Technology Stack
- **React 19** + **TypeScript 5.9** for UI and state management
- **Three.js** for 3D rendering via React Three Fiber (`@react-three/fiber`, `@react-three/drei`)
- **Vite 7** as the build tool with React Compiler for optimization
- **GLTF models** (cake, candle, table) loaded from `/public/*.glb`

## Architecture & Key Components

### Orchestration: `src/App.tsx`
The main orchestrator manages a **choreographed animation sequence** with multiple timed events:
- **Cake descent**: 3-second drop from Y=10 to Y=0 (constants: `CAKE_START_Y`, `CAKE_DESCENT_DURATION`)
- **Table slide**: 0.7-second forward slide, starting 0.1s before cake finishes
- **Candle drop**: 1.2-second fall after cake settles
- **Background fade**: Synchronized fade of the HDR environment (`shanghai_bund_4k.hdr`)

All timing uses interpolation helpers: `lerp()` for linear interpolation and `easeOutCubic()` for acceleration curves. The total animation duration is calculated as `totalAnimationTime`.

### 3D Models: `src/models/`
Each model loads a GLTF file and clones it (important for reusability):
- **Cake** (`cake.tsx`): Loads `/cake.glb`, positioned and animated via parent props
- **Candle** (`candle.tsx`): Uses custom **shader materials** for animated flame (vertex + fragment shaders) with time-based uniforms for flickering. Includes `PointLight` for illumination
- **Table** (`table.tsx`): Loads `/table.glb`
- **PictureFrame** (`pictureFrame.tsx`): Holds reference images

### Interactive Components: `src/components/`
- **BirthdayCard**: 
  - Renders clickable cards with images on the table
  - Manages card hover state with camera focus
  - Responsive to click to toggle active state
  - Uses texture loading with `SRGBColorSpace` for proper color rendering
  - Position/rotation passed from parent (`tablePosition`, `tableRotation` arrays)

- **Fireworks**:
  - Particle-based effect using `BufferGeometry` and `BufferAttribute`
  - Custom updates in `useFrame` for position, velocity, color, and lifetime simulation
  - Configurable `isActive` state and `origin` position
  - Uses gravity simulation (`GRAVITY = -4`)

## Developer Workflows

### Build & Development
```bash
npm install        # Install dependencies
npm run dev        # Start Vite dev server (auto-reload)
npm run build      # Compile TypeScript + build for production
npm run lint       # Run ESLint checks
npm run preview    # Preview production build locally
```

### Key Performance Patterns
- **React Compiler** enabled in Vite config for automatic memoization
- **useFrame hook** used for animation frame updates (replaces `requestAnimationFrame`)
- **Suspense boundaries** wrap Canvas for async GLTF loading
- **useLoader** with GLTFLoader; cloned scenes (`gltf.scene?.clone(true)`) to avoid shared references

## Critical Patterns & Conventions

### Animation Timing
- All animation constants defined at top of `App.tsx` as uppercase `CONST_NAME`
- Use `easeOutCubic()` for natural easing of drops and fades
- Calculate `progress` as normalized value [0, 1] and use `lerp()` for smooth transitions
- Example: `const cakeY = lerp(CAKE_START_Y, CAKE_END_Y, cakeProgress)`

### Three.js Conventions
- **Position arrays** in components: `[x, y, z]` tuples, not `Vector3` objects
- **Group refs** typed as `React.RefObject<Group>` for animation targets
- **Materials**: Use `DoubleSide` for cards/planes visible from both sides
- **Lighting**: `PointLight` for candle flame, `Environment` component for HDR ambient

### State & Callbacks
- Cards managed via `activeCardId` (string) and `cards` array of `BirthdayCardConfig`
- Use `useCallback` for event handlers to maintain referential equality
- `onAnimationComplete` callback fires at end of choreography

## File Organization
```
src/
  App.tsx                    # Main orchestration & animation loop
  components/
    BirthdayCard.tsx         # Interactive card component
    Fireworks.tsx            # Particle effect system
  models/
    cake.tsx, candle.tsx, table.tsx, pictureFrame.tsx  # 3D model wrappers
public/
  shanghai_bund_4k.hdr       # Ambient lighting texture
  cards/                     # HTML-based card templates (if used)
  *.glb                      # GLTF 3D models
```

## Common Tasks

**Adding a new animation element**: Define timing constants at top of `App.tsx`, use `useFrame` + `lerp` + `easeOutCubic` for smooth motion.

**Modifying card behavior**: Edit `BirthdayCard.tsx` - props flow from App (`tablePosition`, `tableRotation`, `isActive`).

**Adjusting timing**: Change constants (e.g., `CAKE_DESCENT_DURATION`) - they propagate through `lerp` calculations.

**Shader updates**: Modify vertex/fragment shaders in `candle.tsx`; update `FlameUniforms` type and `useFrame` uniform assignment.
