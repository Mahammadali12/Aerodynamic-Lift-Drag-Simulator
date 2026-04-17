# Aerodynamic Lift & Drag Simulator

A real-time, interactive physics simulation of aerodynamic forces acting on an airfoil-approximated body, implemented from scratch in C using [raylib](https://www.raylib.com/). All physics are computed analytically — no game engine physics engine is used.

---

## Physics Model

The simulation models four aerodynamic and mechanical forces per timestep, accumulated as accelerations and integrated using a **semi-implicit Euler scheme**:

### 1. Aerodynamic Drag (Parasitic)
$$F_D = \frac{1}{2} \rho C_D A v^2$$

- $\rho = 1.225\ \text{kg/m}^3$ (sea-level air density)
- $C_D = 0.38$ (drag coefficient)
- $A = \pi r^2$ (projected circular cross-section)
- Direction: opposite to velocity vector

### 2. Lift Force
$$F_L = \frac{1}{2} \rho C_L A v^2, \quad C_L = C_{L,\max} \cdot \sin(2\alpha)$$

- Based on thin-airfoil theory; lift scales with $\sin(2\alpha)$
- $\alpha$ = angle of attack (user-controlled, clamped at stall: $\alpha \in [-20°, 45°]$)
- Direction: perpendicular to velocity vector (rotated 90°)

### 3. Induced Drag
$$C_{D,i} = \frac{C_L^2}{\pi \cdot AR \cdot e}, \quad F_{D,i} = \frac{1}{2} \rho C_{D,i} A v^2$$

- $AR = 1.0$ (aspect ratio), $e = 0.9$ (Oswald efficiency factor)
- Increases with angle of attack — models the cost of generating lift

### 4. Ground Friction
$$F_f = \mu \cdot m \cdot g, \quad \mu = 0.7$$

- Applied only in resting/sliding contact with the ground
- Friction reversal prevention: clamps deceleration to avoid velocity flip

---

## Numerical Integration

Semi-implicit Euler (also called symplectic Euler):

```
velocity += acceleration * dt   // update velocity first
position += velocity * dt       // then use new velocity for position
```

This method conserves energy better than explicit Euler and avoids artificial energy injection in oscillating systems (e.g., bouncing).

---

## Controls

| Key | Action |
|-----|--------|
| `A` / `D` | Apply horizontal thrust (left / right) |
| `W` / `S` | Increase / decrease angle of attack (1°/frame) |

---

## Build & Run

**Dependencies:** raylib (linked via your system or vcpkg)

```bash
# Linux / macOS (with raylib installed)
gcc main.c -o sim -lraylib -lm -Wall

# Windows (MinGW)
gcc main.c -o sim.exe -lraylib -lopengl32 -lgdi32 -lwinmm -lm
```

Then run:
```bash
./sim
```

---

## Key Implementation Details

- All physics computed in **SI units** (meters, kg, Newtons, seconds); converted to pixels only at render time via `PPM = 50 px/m`
- Forces accumulated as accelerations each frame via `F/m`; accumulator reset after integration
- Resting contact detection uses a **predicted position check** before integration to set `onGround` state
- Horizontal velocity sleeping: velocities below `0.01 m/s` on ground are zeroed to prevent jitter
- Bounce restitution coefficient: `0.9` (10% energy loss per ground contact)
- Stall modeled by clamping $\alpha$: exceeding $45°$ gives no additional lift (sin function plateau)

---

## What This Demonstrates

- Numerical simulation using physics-based force accumulation
- Aerodynamic force modeling (lift, parasitic drag, induced drag)
- Euler integration with stability considerations
- Real-time interactive simulation loop (60 FPS target)
- Contact/collision handling with resting state and friction
