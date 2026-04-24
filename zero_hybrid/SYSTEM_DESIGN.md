# ZERO HYBRID v5.0 - COMPLETE SYSTEM DESIGN

## Architecture Overview

### 🎯 Core Components

#### 1. **Motion Detection Engine**
- **Pure Frame Differencing**: Only moving pixels trigger detection
- **Adaptive Thresholding**: Brightness target 127 (middle gray)
- **Oscillation Filtering**: Removes 0.5-2 sec UI overlay patterns
- **Auto Kernel Sizing**: Adapts morphological kernel based on motion density
- **Status**: Completely independent of static background

#### 2. **Fish Classification (10+ Classes)**
- **Class 0**: Tiny (< 50 px²) - 1-shot kill
- **Class 1**: Very Small (50-150 px²) - 1 shot
- **Class 2**: Small (150-300 px²) - 1-2 shots
- **Class 3**: Medium-Small (300-600 px²) - 2 shots
- **Class 4**: Medium (600-1000 px²) - 3 shots
- **Class 5**: Medium-Large (1000-1800 px²) - 3-4 shots
- **Class 6**: Large (1800-3000 px²) - 4+ shots
- **Class 7**: Very Large (3000-5000 px²) - Heavy
- **Class 8**: Huge (5000-8000 px²) - Very Heavy
- **Class 9**: Massive (>8000 px²) - Boss

#### 3. **Trajectory Clustering**
- **Fragment Merging**: Same heading, speed, distance = same fish
- **Single Dot Per Fish**: One center point per target
- **Velocity Estimation**: Smooth motion vector from history
- **Lead Calculation**: Predicts impact point 3-4 frames ahead

#### 4. **OCR Multiplier Learning**
- **Score Change Detection**: Reads before/after scores
- **Multiplier Calculation**: (Score Delta) / (Shot Cost)
- **Per-Class Learning**: Stores average multiplier for each class
- **Confidence Metric**: 0 at 1 kill, increases to 1.0 at 10+ kills
- **Persistent Storage**: fish_multipliers.json

#### 5. **Bullet Conservation Manager**
- **Shot Tracking**: Records every shot and cost
- **Kill Tracking**: Confirms fish eliminations
- **Efficiency Ratio**: Points Per Shot metric
- **Cost Management**: 1.0 cost per shot (configurable)
- **Per-Class Stats**: Tracks shots/kills by fish class

#### 6. **Priority Selector**
- **Small Fish First**: Classes 0-2 prioritized
- **Expected Value = Base × Multiplier × Confidence**
- **Score Threshold**: Must exceed min_efficiency × shot_cost
- **Mandatory Fire Window**: If 30 seconds pass, fire at highest confidence
- **Aggressiveness Scaling**: Adjusts threshold based on setting

#### 7. **Aggressiveness Control**
- **+ Key**: Increase aggressiveness (lower threshold, more shots)
- **- Key**: Decrease aggressiveness (higher threshold, fewer shots)
- **Range**: 0.0 (conservative) to 1.0 (aggressive)
- **Effect on Min Efficiency**:
  - Conservative: 5.0x minimum (only obvious kills)
  - Balanced: 2.0x minimum
  - Aggressive: 0.5x minimum (opportunistic)

#### 8. **Adaptive Radar Display**
- **Standard Mode**: Color by fish class
- **Heatmap Mode**: Color by speed (red=fast, blue=slow)
- **Class Only**: Size proportional to class
- **Value Only**: Brightness indicates expected value
- **Mode Cycling**: Press 'R' to switch modes

### 📊 Data Flow

```
Frame Capture
    ↓
Motion Detection (frame diff)
    ↓
Oscillation Detection & Suppression
    ↓
Contour Finding
    ↓
Fish Tracking & Classification (10 classes)
    ↓
Trajectory Clustering (merge fragments)
    ↓
Priority Selection (small fish first)
    ↓
OCR Multiplier Lookup
    ↓
Expected Value Calculation
    ↓
Efficiency Check
    ↓
Fire Decision
    ↓
Lead Calculation & Click
    ↓
Score Recording → Multiplier Update
```

### 🔫 Firing Logic

```
For each fish in priority order:
  1. Get class ID (0-9)
  2. Look up learned multiplier
  3. Calculate base value (size-dependent)
  4. expected_value = base × multiplier
  5. confidence = (multiplier_confidence + kill_rate) / 2
  6. score = expected_value × confidence
  
  IF score > (min_efficiency × shot_cost) × (1 - aggressiveness):
    → Calculate lead (4 frames ahead)
    → Fire shot
    → Deduct cost from score
    → Record to tracker
  
  Every 30 seconds:
    → Fire on best target regardless of threshold
```

### 💾 Persistent Learning

**fish_multipliers.json**
```json
{
  "0": {"avg": 2.5, "confidence": 0.95, "encounters": 50},
  "1": {"avg": 3.0, "confidence": 0.90, "encounters": 45},
  ...
}
```

### ⚙️ Configuration (config.json)

- Motion thresholds
- Fish class boundaries
- Target priorities
- Efficiency minimums
- Oscillation detection parameters
- UI display options

## 🎯 Design Principles

### 1. **Efficiency First**
- Every shot must have positive expected value
- Aggressiveness trades off accuracy for volume
- Goal: Maximize (Points / Shots)

### 2. **Small Fish Priority**
- Least shots to kill = highest efficiency
- Accumulate multiplier data faster
- Build confidence in learning model

### 3. **Adaptive Learning**
- First game: Conservative, learning mode
- As multipliers learned: More aggressive
- Remember patterns across sessions

### 4. **Robust to Environment**
- Motion-only detection ignores static backgrounds
- Works on any game/application
- Oscillation filtering removes UI noise
- Auto-tuning for lighting variations

### 5. **Safety & Control**
- Always require positive expected value
- Aggressiveness is user-controlled
- Mandatory fire window prevents standing idle
- Kill confirmation before credit

## 🚀 Usage

**Basic Operation**
```bash
python zero_hybrid/main_production.py
```

**Controls**
```
Q           → Quit
+ or =      → More aggressive (lower threshold)
- or _      → More conservative (higher threshold)
R           → Cycle radar display modes
```

**Monitoring**
- Main window: Detections with velocity vectors
- Radar window: Fish positions color-coded
- Stats window: Efficiency metrics and learning progress

## 📈 Performance Goals

- **Efficiency**: > 1.5x points per shot (must beat the cost)
- **Accuracy**: 80%+ hit rate on small fish
- **Learning**: Confident multipliers within 10 kills per class
- **Responsiveness**: 30+ FPS minimum
- **Adaptability**: Works on first run with any game
