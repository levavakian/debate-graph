# Debate Graph

A real-time collaborative debate visualization tool that allows users to create, connect, and evaluate logical arguments using graph-based structures. The application supports both desktop and mobile interfaces and uses WebRTC for peer-to-peer collaboration.

## Conceptual Overview

The Debate Graph application is designed to help users visualize logical arguments as interconnected nodes in a graph structure. Users can create statements (premises and conclusions), logical aggregators (AND, OR, XOR, etc.), and connections between them to build complex logical arguments. The system automatically evaluates the validity of statements based on their logical dependencies.

### Core Concepts

1. **Statements**: Text-based nodes representing premises, conclusions, or claims
2. **Aggregators**: Logical operator nodes that combine multiple inputs (AND, OR, XOR, NAND, NOR, IFF)
3. **Connections**: Directed edges between nodes that can carry unary operators (NOT) or binary operators (IF_THEN)
4. **Validity Evaluation**: Automatic logical evaluation that propagates truth values through the graph
5. **Real-time Collaboration**: Multiple users can work on the same debate graph simultaneously

## Technical Architecture

### Frontend Stack
- **HTML5/CSS3/JavaScript (ES6+)**: Core web technologies
- **D3.js v7.8.5**: Graph visualization and force simulation
- **Trystero v0.21.3**: WebRTC peer-to-peer networking library
- **No build tools**: Direct browser execution with ES6 modules

### Networking Architecture
- **WebRTC**: Peer-to-peer real-time communication
- **Trystero**: Abstraction layer over WebRTC for simplified P2P connections
- **Host-Client Model**: One user acts as host, others join as clients
- **State Synchronization**: All changes broadcast from host to all peers

### Data Structures

```javascript
// Global State
state = {
    statements: {
        [id]: {
            id: string,
            text: string,
            x: number,
            y: number,
            validity: boolean|null,
            createdBy: string
        }
    },
    aggregators: {
        [id]: {
            id: string,
            operator: string, // 'AND', 'OR', 'XOR', 'NAND', 'NOR', 'IFF'
            x: number,
            y: number,
            validity: boolean|null,
            createdBy: string
        }
    },
    connections: {
        [id]: {
            id: string,
            from: string, // source node id
            to: string,   // target node id
            operator: string|null, // 'NOT', 'IF_THEN', or null for plain connections
            createdBy: string
        }
    }
}
```

## User Interface Design

### Main Canvas (Primary Interface)

**Layout**: Full-screen SVG canvas with floating control panel

**Desktop Features**:
- **Canvas**: Infinite scrollable/pannable workspace
- **Node Interaction**: 
  - Double-click empty space: Create statement
  - Ctrl+click empty space: Create aggregator (shows type selector)
  - Shift+drag between nodes: Create connection
  - Double-click statement: Edit text inline
  - Right-click any element: Delete
  - Drag nodes: Reposition
- **Visual Elements**:
  - Statements: White rectangles with black text, colored borders (green=valid, red=invalid, gray=undetermined)
  - Aggregators: Blue rectangles with operator name and symbol (e.g., "AND (∧)")
  - Connections: Arrows with optional operator symbols for unary operations
  - Drag line: Dashed line during connection creation

**Mobile Adaptations**:
- Touch gestures replace mouse interactions
- Long press replaces right-click
- Tap and hold + drag for connections
- Pinch to zoom, two-finger pan
- Larger touch targets for mobile fingers

### Control Panel (Fixed Position)

**Location**: Top-left corner, always visible

**Components**:
1. **Room Management Section**:
   - "Create Room" button → generates 6-character room code
   - "Join Room" input field + "Join Room" button
   - Room info display (code + connection status)

2. **Content Creation Section**:
   - "Add Statement" button → creates statement at canvas center
   - "Add Aggregator" button → shows aggregator type selector near button

3. **Data Management Section**:
   - "Export Debate" button → opens modal with JSON data
   - "Import Debate" button → file picker for JSON import

4. **Instructions Section**:
   - Compact help text with keyboard shortcuts
   - Responsive text that adapts to mobile

### Modal Dialogs

#### Aggregator Type Selector
**Trigger**: "Add Aggregator" button or Ctrl+click canvas
**Layout**: Small popup with operator buttons
**Options**: AND (∧), OR (∨), XOR (⊕), NAND (⊼), NOR (⊽), IFF (↔)
**Positioning**: Near trigger button or at click location

#### Connection Operator Selector
**Trigger**: Completing shift+drag between nodes
**Layout**: Small popup with operator buttons
**Options**: IF → THEN, NOT (¬), Plain Connection
**Positioning**: At mouse release location

#### Export Modal
**Trigger**: "Export Debate" button
**Layout**: Full-screen modal with textarea
**Features**: 
- JSON data display (read-only)
- "Copy to Clipboard" button
- "Download as File" button
- "Close" button

#### Statement Editor
**Trigger**: Double-click statement
**Layout**: Inline text input at statement location
**Features**:
- Pre-filled with current text
- Enter to save, Escape to cancel
- Auto-focus and select all text

### Context Menus

#### Right-click Menu
**Trigger**: Right-click any graph element
**Options**: "Delete" (only option currently)
**Positioning**: At cursor location

### Toast Notifications

**Purpose**: User feedback for actions and errors
**Examples**: 
- "Copied to clipboard!"
- "Debate imported successfully!"
- "Cannot create connection: would cause circular dependency"
**Styling**: Bottom-center, auto-dismiss after 3 seconds

## Logical Evaluation System

### Evaluation Rules

1. **Axioms**: Statements with no incoming connections are considered true
2. **Aggregators**: Apply their operator to all incoming values
   - AND: All inputs must be true
   - OR: At least one input must be true
   - XOR: Exactly one input must be true
   - NAND: NOT(AND) - not all inputs are true
   - NOR: NOT(OR) - no inputs are true
   - IFF: All inputs have the same value
3. **Unary Operators**: Applied to connection values
   - NOT: Inverts the boolean value
4. **Binary Operators**: Applied between nodes
   - IF_THEN: Logical implication (!A || B)

### Evaluation Algorithm

1. **Reset**: Set all validities to null
2. **Identify Axioms**: Find nodes with no incoming connections
3. **Topological Sort**: Order nodes by dependency
4. **Propagate**: Evaluate each node based on its inputs and operators
5. **Cycle Detection**: Prevent circular dependencies

### Visual Feedback

- **Valid (True)**: Green border and light green background
- **Invalid (False)**: Red border and light red background  
- **Undetermined**: Gray border and light gray background

## Collaboration Features

### Room Management

**Room Creation**:
1. User clicks "Create Room"
2. System generates 6-character alphanumeric code
3. User becomes host
4. Room code displayed for sharing

**Room Joining**:
1. User enters room code
2. Clicks "Join Room"
3. Connects to host via WebRTC
4. Receives current state from host

### Real-time Synchronization

**Action Broadcasting**:
- All user actions converted to action objects
- Host processes actions and broadcasts to all peers
- Clients receive and apply actions to local state

**Action Types**:
- ADD_STATEMENT, UPDATE_STATEMENT, DELETE_STATEMENT
- ADD_AGGREGATOR, UPDATE_AGGREGATOR, DELETE_AGGREGATOR  
- ADD_CONNECTION, DELETE_CONNECTION

**Conflict Resolution**:
- Host authority model (host's actions take precedence)
- Timestamp-based IDs prevent conflicts
- No operational transformation needed due to simple action model

### Connection Management

**Status Display**: Shows number of connected peers
**Peer Handling**: Automatic reconnection attempts
**State Recovery**: New peers receive full state on join

## Force Simulation

### D3.js Physics

**Forces Applied**:
- **Link Force**: Connects related nodes with springs (distance: 150px)
- **Charge Force**: Repulsion between nodes (strength: -300)
- **Center Force**: Pulls nodes toward canvas center
- **Collision Force**: Prevents node overlap (radius: 80px)

**Node Positioning**:
- Fixed positions (fx, fy) to prevent unwanted movement
- Manual dragging updates fixed positions
- Simulation restarts on graph changes

## Mobile Responsiveness

### Touch Interactions

**Gesture Mapping**:
- Tap → Click
- Double-tap → Double-click
- Long press → Right-click
- Pinch → Zoom
- Two-finger drag → Pan
- Single finger drag → Move node
- Shift+drag simulation → Long press + drag

### Layout Adaptations

**Control Panel**:
- Larger buttons for touch targets (minimum 44px)
- Simplified layout on small screens
- Collapsible sections if needed

**Canvas**:
- Touch-friendly zoom controls
- Larger node sizes on mobile
- Simplified visual effects for performance

### Performance Considerations

**Mobile Optimizations**:
- Reduced particle count in force simulation
- Simplified animations
- Efficient touch event handling
- Memory management for large graphs

## File Format

### Export/Import Structure

```json
{
    "statements": {
        "stmt_123": {
            "id": "stmt_123",
            "text": "The sky is blue",
            "x": 400,
            "y": 300,
            "validity": true,
            "createdBy": "user_abc"
        }
    },
    "aggregators": {
        "agg_456": {
            "id": "agg_456", 
            "operator": "AND",
            "x": 600,
            "y": 300,
            "validity": null,
            "createdBy": "user_abc"
        }
    },
    "connections": {
        "conn_789": {
            "id": "conn_789",
            "from": "stmt_123",
            "to": "agg_456", 
            "operator": null,
            "createdBy": "user_abc"
        }
    }
}
```

## Error Handling

### Validation Rules

**Circular Dependency Prevention**:
- Depth-first search cycle detection
- Prevents connections that would create loops
- User notification via toast

**Input Validation**:
- Non-empty statement text required
- Valid room codes (6 alphanumeric characters)
- File format validation for imports

### User Feedback

**Error Messages**:
- Clear, actionable error descriptions
- Toast notifications for temporary errors
- Modal dialogs for critical errors

## Browser Compatibility

### Requirements

**Minimum Support**:
- ES6 modules support
- WebRTC support
- SVG support
- Modern event handling

### Fallbacks

**Progressive Enhancement**:
- Core functionality works without WebRTC (local mode)
- Graceful degradation for older browsers
- Feature detection for advanced capabilities
