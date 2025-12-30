# Kryten WebUI Requirements

## Overview

A modern, clean web interface for monitoring and managing the Kryten microservice ecosystem. The WebUI provides real-time visibility into service health, event streams, logs, configuration, and user metrics through a responsive, "snazzy" interface. Think of it as a **mission control dashboard** for your Kryten services.

## Core Purpose

The WebUI serves as a centralized monitoring and management console that:
- **Discovers** all services in the ecosystem automatically via NATS
- **Visualizes** real-time health, heartbeats, and service topology
- **Streams** live NATS events with powerful filtering
- **Aggregates** logs from all services in one view
- **Displays** service configurations for debugging
- **Tracks** user activity and statistics via kryten-userstats
- **Controls** service lifecycle (restart, reload config, shutdown)

## Authentication & Authorization

### Login Flow

1. **Username Entry**: User enters their CyTube username
2. **Permission Check**: Verify user is moderator or above in CyTube
3. **One-Time Code**: System PMs the user a one-time code via CyTube
4. **Code Verification**: User enters code in WebUI to complete authentication
5. **Session Management**: Maintain secure session with appropriate timeout

### Authorization Levels

- **Moderator**: Read-only monitoring (service health, events, logs, user stats)
- **Admin**: Full access including service control (restart, reload config, shutdown)
- **Owner**: All admin privileges plus critical operations (shutdown kryten-robot)

### Security Requirements

- Secure session tokens with expiration
- Rate limiting on authentication attempts
- Code expiration (e.g., 5 minutes)
- Secure WebSocket connections (WSS)
- CORS configuration for API endpoints
- Audit logging for all service control actions

## Core Features

### 1. Service Discovery & Topology

#### Service Registry Dashboard

- **Auto-Discovery**: Services automatically discovered via `kryten.service.discovery` and `kryten.service.heartbeat` subjects
- **Service Cards**: Visual cards for each discovered service showing:
  - Service name and version
  - Current health status (healthy/degraded/failing/down)
  - Uptime since last startup
  - Last heartbeat timestamp
  - Hostname/deployment location
  - Quick action buttons (restart, reload config, view logs)
- **Status Indicators**:
  - 🟢 Healthy (green) - Recent heartbeat, status "healthy"
  - 🟡 Degraded (yellow) - Recent heartbeat, status "degraded"
  - 🔴 Failing (red) - Recent heartbeat, status "failing"
  - ⚫ Down (gray) - Missing heartbeats (no heartbeat in 30+ seconds)
  - 🔵 Starting (blue) - Startup event received, awaiting first heartbeat

#### Service Topology Visualization

- **Interactive Graph**: Visual representation of service interconnections
  - Nodes for each service (size proportional to activity/importance)
  - Edges showing communication patterns (derived from event flow)
  - Color-coded by health status
  - Click to drill into service details
- **Connection Map**: Shows how services interact
  - Kryten-Robot as central hub
  - Service-to-service communication patterns
  - Event flow visualization
- **Health Heatmap**: Grid view of all services with color-coded health

#### Service Details View

Clicking a service card opens detailed view:
- **Lifecycle Timeline**: Visual timeline of startup, connections, disconnections, shutdowns
- **Heartbeat History**: Chart showing heartbeat consistency over time
- **Health Metrics**: Service-specific metrics (if exposed)
  - Message throughput
  - Error rates
  - Response times
  - Custom metrics per service
- **Configuration Snapshot**: Current service configuration (via `get_config()`)
- **Recent Events**: Last 100 events published by this service
- **Log Tail**: Last 200 lines of service logs
- **Control Panel**: Admin-only buttons for restart, reload config, shutdown

### 2. Real-Time Event Stream Viewer

#### Live Event Dashboard

- **Event Stream**: Real-time scrolling feed of all NATS events
  - Subject pattern (e.g., `kryten.events.cytube.420grindhouse.chatmsg`)
  - Timestamp (precise to millisecond)
  - Service source (which service published it)
  - Event type (chatmsg, adduser, userleave, etc.)
  - Payload preview (expandable JSON)
  - Color-coded by event category
- **Powerful Filtering**:
  - By subject pattern (supports wildcards: `kryten.events.*.*.chatmsg`)
  - By service name (e.g., show only robot events)
  - By event type (chat, user, playlist, lifecycle, etc.)
  - By time range (last 5 min, 1 hour, 24 hours, custom)
  - By search text in payload
  - Multiple filters combinable with AND/OR logic
- **Event Inspector**: Click any event for detailed view
  - Full JSON payload with syntax highlighting
  - Correlated events (events with same correlation_id)
  - Event trace (follow event through the system)
  - Export individual event as JSON

#### Event Statistics

- **Event Rate Graphs**: Real-time charts showing:
  - Events per second (overall and per service)
  - Event type distribution (pie chart)
  - Top 10 most active subjects
  - Error event rate (track system health)
- **Event Analytics**:
  - Total events processed (lifetime counter)
  - Peak event rate (events/sec)
  - Average event size
  - Event latency (if correlation_id tracking available)

#### Event Recording & Playback

- **Session Recording**: Capture event stream to file for later analysis
- **Event Export**: Export filtered events as JSON, CSV, or JSONL
- **Event Search**: Full-text search across recorded event history
- **Replay Mode**: Playback recorded events at original speed or faster

### 3. Service Log Aggregation

#### Unified Log Viewer

- **Real-Time Logs**: Live log tail from all services in single view
  - Service name/source prominently displayed
  - Log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  - Timestamp
  - Log message with syntax highlighting
  - Stack traces (expandable for errors)
- **Log Filtering**:
  - By service name
  - By log level
  - By time range
  - By search text (supports regex)
  - Show only errors/warnings (quick filter)
- **Log Statistics**:
  - Error rate per service
  - Warning rate over time
  - Most frequent log messages
  - Services with highest log volume

#### Service-Specific Log View

- **Dedicated Log Panel**: View logs for single service
  - Tail mode (auto-scroll with new logs)
  - Historical mode (scroll through past logs)
  - Download logs as text file
  - Share log permalink (specific time range)

### 4. Configuration Viewer

#### Service Configuration Dashboard

- **Configuration Browser**: View current config for each service
  - kryten-robot config (via existing `get_config()` method)
  - Other services (requires extension - expose config via NATS)
  - Passwords/secrets redacted automatically
  - JSON syntax highlighting
- **Configuration Comparison**:
  - Compare current config vs. default/example
  - Highlight differences
  - Show config drift warnings
- **Configuration History** (future):
  - Track config changes over time
  - Diff view between versions
  - Rollback capability

### 5. User Metrics & Statistics

#### User Activity Dashboard

Integration with **kryten-userstats** service:

- **Active Users Panel**:
  - Currently connected users (live list)
  - User rank badges (owner, admin, moderator, regular)
  - Real-time activity indicators
  - Click user for detailed stats
- **User Statistics**:
  - **Per-User Stats**:
    - Total chat messages
    - Videos added to playlist
    - Time in channel (total and average per session)
    - Last seen timestamp
    - Activity graph (messages over time)
    - Known aliases
  - **Global Channel Stats**:
    - Total unique users (lifetime)
    - Most active users (top 10/20)
    - Message rate over time
    - Peak concurrent users
    - Retention metrics
- **User Analytics**:
  - User join/leave rate
  - User engagement trends
  - Hourly/daily/weekly activity patterns
  - User retention graphs
  - Popular times for channel activity

#### User Profile Inspector

Click any user to see:
- Detailed statistics
- Activity timeline
- Message history (if logged)
- Videos added history
- Moderation history
- Known aliases
- First seen / last seen

### 6. Service Control & Management

#### Service Lifecycle Control

**Admin+ Only** - Control panel for each service:

- **Restart**: Gracefully restart individual service
  - Uses `kryten.lifecycle.group.restart` with service filter
  - Confirm dialog with reason field
  - Shows restart progress (shutdown → startup)
- **Reload Config**: Trigger config reload for service
  - kryten-robot: via existing `reload_config()` method
  - Other services: via service-specific command subjects
  - Confirm dialog
- **Shutdown**: Gracefully stop service
  - kryten-robot: via existing `shutdown(delay_seconds, reason)` method
  - Other services: via service-specific shutdown commands
  - Confirm dialog with delay and reason fields
  - Owner-only for kryten-robot shutdown
- **Health Check**: Force immediate health check/heartbeat
  - Useful for testing service responsiveness

#### System-Wide Operations

- **Group Restart**: Restart all services in coordinated fashion
  - Uses existing `kryten.lifecycle.group.restart` broadcast
  - Specify restart delay (5-60 seconds)
  - Confirm dialog with reason
- **Service Ordering**: Define restart order dependencies (future)

### 7. Health & Metrics Dashboard

#### System Health Overview

- **Health Summary Card**:
  - Total services discovered
  - Healthy / degraded / failing / down counts
  - Overall system health score (percentage)
  - Alert count (critical issues requiring attention)
- **Health Timeline**: Historical graph of system health over time
- **Alert Panel**: Active alerts/warnings
  - Service down alerts
  - Repeated failures
  - Missing heartbeats
  - High error rates
  - Configurable thresholds

#### Service Metrics (If Exposed)

For services that publish metrics:
- **Performance Metrics**:
  - CPU usage
  - Memory usage
  - Message queue depth
  - Request throughput
  - Response time percentiles (p50, p95, p99)
- **Business Metrics**:
  - Commands processed
  - Events published
  - Errors encountered
  - Custom service-specific metrics

### 8. Analytics & Insights

#### System Analytics Dashboard

- **Event Flow Analysis**:
  - Most common event paths through system
  - Event flow diagrams (Sankey charts)
  - Bottleneck identification
- **Service Performance**:
  - Service uptime percentage
  - Mean time between failures (MTBF)
  - Mean time to recovery (MTTR)
  - Service reliability trends
- **Historical Reports**:
  - Daily/weekly/monthly summaries
  - Service availability reports
  - Event volume trends
  - Error rate trends
  - Exportable reports (PDF, CSV, JSON)

## Out of Scope

The following features are **explicitly excluded** from this WebUI:

- ❌ **Playlist Management**: No queue manipulation, reordering, or video control
- ❌ **Video Playback**: No video player, play/pause/seek controls
- ❌ **Chat Interface**: No sending chat messages or viewing live chat
- ❌ **Broadcast Messaging**: No mass PM functionality
- ❌ **User Moderation**: No kick/ban/mute actions (use dedicated bot or CyTube UI)
- ❌ **CyTube Interaction**: Focus is on Kryten ecosystem, not CyTube channel management

## UI/UX Design Requirements

### Visual Design

- **Modern Aesthetic**:
  - Clean, minimalist layout inspired by monitoring tools (Grafana, DataDog, New Relic)
  - Ample whitespace
  - Consistent color scheme
  - Professional typography (monospace for logs/events)
- **Color Palette**:
  - Dark mode as primary theme (easier on eyes for monitoring)
  - Light mode option available
  - Semantic colors:
    - 🟢 Green: Healthy, success
    - 🟡 Yellow: Degraded, warnings
    - 🔴 Red: Failing, errors, critical
    - 🔵 Blue: Info, starting
    - ⚫ Gray: Down, unknown, disabled
  - Brand colors for Kryten ecosystem (purple/cyan accents)
- **Responsive Design**:
  - Desktop-first (monitoring tool)
  - Tablet support for viewing
  - Mobile view for quick status checks

### User Experience

- **Intuitive Navigation**:
  - Sidebar navigation with icons
  - Breadcrumbs for deep navigation
  - Quick access toolbar with global actions
  - Keyboard shortcuts for power users
- **Real-Time Updates**:
  - WebSocket-driven updates (no polling)
  - Smooth animations for status changes
  - Loading states for async operations
  - Optimistic UI updates where appropriate
- **Performance**:
  - Fast initial load (< 2 seconds)
  - Lazy loading for heavy components (logs, events)
  - Virtual scrolling for long lists (1000+ items)
  - Efficient re-rendering (React.memo, useMemo)
  - Minimal network overhead

### Component Library

- **Framework**: React + TypeScript
- **Styling**: Tailwind CSS for utility-first approach
- **Visualization**:
  - Recharts or Chart.js for graphs
  - D3.js for topology visualization
  - React Flow for service connection diagrams
- **Components**:
  - Modals/dialogs (service control confirmations)
  - Toast notifications (alerts, success messages)
  - Data tables with sorting/filtering/pagination
  - Progress indicators and spinners
  - Status badges
  - Action buttons
  - Form inputs with validation
  - Code/JSON viewers with syntax highlighting

### "Snazzy" Elements

- **Animations**:
  - Smooth transitions between views
  - Micro-interactions (button hovers, clicks)
  - Pulsing status indicators for real-time data
  - Loading animations (skeleton loaders)
  - Service cards fade in on discovery
- **Visual Feedback**:
  - Success/error animations for actions
  - Confirmation dialogs with nice styling
  - Real-time metric counters (count-up animations)
  - Pulsing heartbeat indicators
  - Connection status animations
- **Modern Features**:
  - Glassmorphism styling for cards
  - Gradient accents
  - Icon animations
  - Smooth scrolling
  - Sticky headers
  - Collapsible sections

## Technical Architecture

### Frontend Stack

- **Framework**: React 18+ with TypeScript
- **State Management**: Zustand (lightweight, simple)
- **Routing**: React Router v6
- **WebSocket**: Native WebSocket API or Socket.io-client
- **HTTP Client**: Axios for REST API calls
- **Build Tool**: Vite (fast, modern)
- **Testing**: Vitest + React Testing Library

### Backend/API Layer

- **Web Framework**: FastAPI (Python 3.11+)
- **WebSocket Server**: FastAPI WebSocket support or Socket.io
- **NATS Integration**: kryten-py library for ALL service cluster communication
- **Authentication**: JWT tokens (access + refresh)
- **Session Management**: Redis for distributed sessions
- **Database**: PostgreSQL for event/log persistence (optional, future)

### Communication Flow

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Browser    │────▶│  WebUI API   │────▶│  kryten-py   │
│  (React)    │◀────│  (FastAPI)   │◀────│ KrytenClient │
└─────────────┘     └──────────────┘     └──────────────┘
      │                    │                      │
      │                    │                      ▼
      │                    │              ┌──────────────┐
      │                    │              │     NATS     │
      │                    │              │   Message    │
      │                    │              │     Bus      │
      │                    │              └──────────────┘
      │                    │                      │
      │                    │          ┌───────────┴────────────┐
      │                    │          │                        │
      │                    │     ┌────▼─────┐          ┌──────▼──────┐
      │                    │     │ Kryten   │          │   Other     │
      │                    │     │  Robot   │          │  Services   │
      │                    │     └──────────┘          └─────────────┘
      │                    │
      └────WebSocket───────┘
       (Real-time updates)
```

### Service Discovery Architecture

#### NATS Subject Structure

**Service Registration** (one-time on startup):
- Subject: `kryten.service.discovery`
- Published by: Each service on startup
- Payload:
```json
{
  "service": "kryten-llm",
  "version": "0.2.1",
  "hostname": "server1.example.com",
  "capabilities": ["llm.query", "llm.summarize"],
  "command_subject": "kryten.llm.command",
  "timestamp": "2025-12-09T15:30:00Z"
}
```

**Service Heartbeat** (periodic, every 5-10 seconds):
- Subject: `kryten.service.heartbeat`
- Published by: Each service periodically
- Payload:
```json
{
  "service": "kryten-llm",
  "status": "healthy",
  "uptime_seconds": 3600,
  "hostname": "server1.example.com",
  "timestamp": "2025-12-09T15:30:05Z",
  "metrics": {
    "messages_processed": 1234,
    "errors": 2,
    "queue_depth": 5
  }
}
```

**Status Values**:
- `healthy` - Service operating normally
- `degraded` - Service operational but with issues (high latency, some errors)
- `failing` - Service experiencing significant issues but still running

**Service De-registration** (graceful shutdown):
- Uses existing `kryten.lifecycle.{service}.shutdown` event
- WebUI marks service as "down" after missing N heartbeats (default: 3 missed = 30 seconds)

**Re-registration Broadcast**:
- Subject: `kryten.service.discovery.poll`
- Published by: kryten-robot or WebUI admin
- Payload: `{"reason": "Forced re-registration"}`
- Effect: All services republish to `kryten.service.discovery`
- Use cases:
  - kryten-robot restart
  - Forcing service re-discovery from WebUI
  - Also triggered by `kryten.lifecycle.robot.startup` event

#### Service Registry in Kryten-Robot

**KV Store Schema** (NATS JetStream KV bucket: `kryten_services`):

```
Key: "registry:{service_name}"
Value: {
  "service": "kryten-llm",
  "version": "0.2.1",
  "hostname": "server1.example.com",
  "capabilities": ["llm.query"],
  "command_subject": "kryten.llm.command",
  "first_seen": "2025-12-09T15:00:00Z",
  "last_heartbeat": "2025-12-09T15:30:05Z",
  "heartbeat_count": 360,
  "status": "healthy"
}

Key: "heartbeat:{service_name}"
Value: {
  "status": "healthy",
  "timestamp": "2025-12-09T15:30:05Z",
  "uptime_seconds": 3600,
  "metrics": {...}
}
```

**Kryten-Robot Responsibilities**:
1. Subscribe to `kryten.service.discovery` - Store new service registrations in KV
2. Subscribe to `kryten.service.heartbeat` - Update last heartbeat timestamp in KV
3. Subscribe to `kryten.service.discovery.poll` - Broadcast poll request
4. Publish own startup on `kryten.lifecycle.robot.startup` - Triggers service re-registration
5. Expose command: `system.services` - Return list of registered services from KV
6. Periodic cleanup: Mark services as "down" if no heartbeat in 30+ seconds

### Kryten-Py Integration

**CRITICAL**: All communication with the service cluster MUST go through kryten-py's KrytenClient.

#### Current KrytenClient Capabilities (v0.6.0+)

**Lifecycle Events** (existing):
- `LifecycleEventPublisher` class for publishing lifecycle events
- `on_restart_notice(callback)` - Register callback for group restart notices
- Subjects:
  - `kryten.lifecycle.{service}.startup`
  - `kryten.lifecycle.{service}.shutdown`
  - `kryten.lifecycle.{service}.connected`
  - `kryten.lifecycle.{service}.disconnected`
  - `kryten.lifecycle.group.restart`

**System Management** (Kryten-Robot control):
- `get_stats()` - Comprehensive runtime statistics
- `get_config()` - Current configuration (passwords redacted)
- `get_version()` - Kryten-Robot version string
- `get_channels()` - Discover available channels
- `ping()` - Lightweight health check
- `reload_config(config_path=None)` - Reload configuration
- `shutdown(delay_seconds=0, reason="...")` - Graceful shutdown
- `nats_request(subject, request, timeout)` - Generic NATS request/reply

**KeyValue Store** (NATS JetStream):
- `kv_get(bucket, key, default=None, parse_json=False)` - Get value
- `kv_put(bucket, key, value, as_json=False)` - Store value
- `kv_delete(bucket, key)` - Delete value
- `kv_keys(bucket)` - List all keys
- `kv_get_all(bucket, parse_json=False)` - Get all key-value pairs

**Event System**:
- `@client.on(event_name, channel=None, domain=None)` - Decorator for event handlers
- Supports subscribing to any NATS subject pattern

#### Required Extensions to kryten-py

**Phase 0: Core Service Discovery** (CRITICAL):

1. **Service Discovery Subscriptions**:
   - `subscribe_service_discovery(callback: Callable) -> None` - Subscribe to `kryten.service.discovery`
   - `subscribe_heartbeat(callback: Callable) -> None` - Subscribe to `kryten.service.heartbeat`
   - `subscribe_lifecycle_events(event_type: str, callback: Callable) -> None` - Subscribe to lifecycle events

2. **Service Registry Queries** (via Kryten-Robot):
   - `get_registered_services() -> list[dict]` - Query `system.services` command
   - `get_service_info(service_name: str) -> dict` - Get specific service from KV registry
   - `force_service_reregistration(reason: str = "Manual poll") -> None` - Publish to `kryten.service.discovery.poll`

3. **Service Control**:
   - `restart_service(service_name: str, reason: str = "") -> None` - Restart specific service via group restart with filter
   - `broadcast_group_restart(delay_seconds: int = 5, reason: str = "") -> None` - Restart all services (existing subject)
   - `shutdown_service(service_name: str, delay_seconds: int = 5, reason: str = "") -> None` - Shutdown specific service

4. **Event Stream Access**:
   - `subscribe_all_events(callback: Callable, pattern: str = "kryten.>") -> None` - Subscribe to wildcard subject for event streaming
   - `subscribe_filtered_events(callback: Callable, patterns: list[str]) -> None` - Multiple subject patterns

**Phase 1: Log Aggregation** (requires services to implement):

5. **Service Log Access**:
   - `subscribe_logs(service_name: str, callback: Callable) -> None` - Subscribe to `kryten.logs.{service}`
   - `get_log_history(service_name: str, lines: int = 100) -> list` - Get recent logs from KV or service

**Phase 2: User Statistics** (via kryten-userstats):

6. **User Statistics Integration**:
   - `get_user_stats(username: str = None) -> dict` - Query userstats service
   - `get_user_aliases(username: str) -> list` - Query aliases from userstats
   - `get_channel_stats() -> dict` - Aggregate channel statistics
   - `subscribe_user_stats(callback: Callable) -> None` - Real-time stats updates

**Phase 3: Configuration Access**:

7. **Multi-Service Configuration**:
   - `get_service_config(service_name: str) -> dict` - Query config from any service (requires services to implement `config.get` command)

### Data Flow Examples

#### Service Discovery Flow

1. **On Service Startup**:
   - Service publishes to `kryten.service.discovery` with registration payload
   - Kryten-Robot receives, stores in KV bucket `kryten_services:registry:{service}`
   - WebUI backend subscribes via `client.subscribe_service_discovery(callback)`
   - Callback receives registration, adds service to UI registry
   - WebUI displays new service card (fade-in animation)

2. **Periodic Heartbeats**:
   - Every 5-10 seconds, service publishes to `kryten.service.heartbeat`
   - Kryten-Robot updates KV: `kryten_services:heartbeat:{service}`
   - WebUI backend subscribes via `client.subscribe_heartbeat(callback)`
   - Callback updates service status and last heartbeat timestamp
   - UI updates service card status indicator in real-time

3. **WebUI Initialization**:
   - Backend connects to NATS via kryten-py
   - Calls `client.get_registered_services()` to get current service list from kryten-robot KV
   - Displays all known services immediately
   - Subscribes to discovery and heartbeat for real-time updates
   - Within 20 seconds, all services have sent heartbeat (confirms they're alive)

4. **Kryten-Robot Restart**:
   - Kryten-Robot publishes `kryten.lifecycle.robot.startup`
   - All services listening on this subject re-publish to `kryten.service.discovery`
   - Kryten-Robot rebuilds service registry from fresh registrations
   - WebUI receives updated service list

5. **Manual Re-registration**:
   - Admin clicks "Force Service Scan" button in WebUI
   - Backend calls `client.force_service_reregistration("Manual scan")`
   - kryten-py publishes to `kryten.service.discovery.poll`
   - All services republish to `kryten.service.discovery`
   - WebUI receives fresh registration data

#### Service Control Flow

1. **Restart Service**:
   - Admin clicks "Restart" button on service card
   - Confirmation dialog: "Restart kryten-llm? Reason: [input]"
   - Backend calls `client.restart_service("kryten-llm", reason="Manual restart for config change")`
   - kryten-py publishes to `kryten.lifecycle.group.restart` with service filter
   - Target service receives restart notice, begins graceful shutdown
   - Service publishes `kryten.lifecycle.{service}.shutdown`
   - WebUI marks service as "restarting" (blue status)
   - Service restarts, publishes `kryten.lifecycle.{service}.startup`
   - Service resumes heartbeats
   - WebUI status changes to "healthy" (green)

2. **View Service Logs**:
   - User clicks service card, navigates to logs tab
   - Backend subscribes via `client.subscribe_logs("kryten-llm", callback)`
   - Callback receives log lines, stores in buffer (last 1000 lines)
   - Streams logs to frontend via WebSocket
   - UI displays logs with syntax highlighting, auto-scrolls

#### Event Streaming Flow

1. **Real-Time Event Viewer**:
   - User opens Event Stream page
   - Backend subscribes via `client.subscribe_all_events(callback, "kryten.>")`
   - Every NATS event received triggers callback
   - Backend forwards event to frontend via WebSocket
   - UI displays event in scrolling feed
   - Event cards fade in with animation

2. **Filtered Event Stream**:
   - User applies filter: "Show only chatmsg events from 420grindhouse"
   - Frontend sends filter to backend via WebSocket
   - Backend calls `client.subscribe_filtered_events(callback, ["kryten.events.cytube.420grindhouse.chatmsg"])`
   - Only matching events forwarded to UI
   - UI updates in real-time with filtered results

#### Authentication Flow

1. User enters username in WebUI
2. Backend calls `client.get_user(channel, username)` to verify user exists and get rank
3. If rank >= 2 (moderator), generate OTP
4. Backend calls `client.send_pm(channel, username, f"WebUI login code: {otp}")`
5. kryten-py sends PM command → Kryten-Robot → CyTube
6. User receives PM in CyTube, enters OTP in WebUI
7. Backend validates OTP, creates JWT session tokens (access + refresh)
8. Frontend stores tokens, includes in subsequent requests
9. WebUI loads with user's permission level (moderator/admin/owner)

## Configuration

### WebUI Configuration File

```json
{
  "server": {
    "host": "0.0.0.0",
    "port": 8080,
    "ssl_cert": "/path/to/cert.pem",
    "ssl_key": "/path/to/key.pem"
  },
  "nats": {
    "servers": ["nats://localhost:4222"],
    "credentials": "/path/to/nats.creds",
    "max_reconnect_attempts": 10
  },
  "kryten": {
    "channel": "your-channel",
    "domain": "cytu.be",
    "service_name": "webui"
  },
  "auth": {
    "otp_expiration_seconds": 300,
    "session_timeout_hours": 24,
    "jwt_secret": "your-secret-key-here",
    "max_login_attempts": 3,
    "lockout_duration_seconds": 300
  },
  "services": {
    "heartbeat_timeout_seconds": 30,
    "heartbeat_missed_threshold": 3,
    "registry_kv_bucket": "kryten_services"
  },
  "ui": {
    "theme": "dark",
    "event_buffer_size": 1000,
    "log_buffer_size": 1000,
    "max_services_display": 100
  },
  "redis": {
    "host": "localhost",
    "port": 6379,
    "db": 0
  }
}
```

## Development Phases

### Phase 0: Infrastructure (Kryten-Robot & kryten-py)

**Kryten-Robot Extensions**:
- [ ] Subscribe to `kryten.service.discovery` - Store registrations in KV
- [ ] Subscribe to `kryten.service.heartbeat` - Update heartbeat timestamps in KV
- [ ] Subscribe to `kryten.service.discovery.poll` - Trigger re-registration broadcast
- [ ] Implement `system.services` command - Return registered services from KV
- [ ] Implement periodic heartbeat timeout checker (mark services down)
- [ ] Listen to own `kryten.lifecycle.robot.startup` - Broadcast re-registration poll

**kryten-py Extensions**:
- [ ] Add `subscribe_service_discovery(callback)` - Subscribe to service registrations
- [ ] Add `subscribe_heartbeat(callback)` - Subscribe to service heartbeats
- [ ] Add `subscribe_lifecycle_events(event_type, callback)` - Subscribe to lifecycle events
- [ ] Add `get_registered_services()` - Query kryten-robot for service list
- [ ] Add `get_service_info(service_name)` - Get specific service from registry
- [ ] Add `force_service_reregistration(reason)` - Publish poll event
- [ ] Add `restart_service(service_name, reason)` - Restart specific service
- [ ] Add `shutdown_service(service_name, delay, reason)` - Shutdown specific service
- [ ] Add `broadcast_group_restart(delay, reason)` - Restart all services
- [ ] Add `subscribe_all_events(callback, pattern)` - Wildcard event subscription
- [ ] Add `subscribe_filtered_events(callback, patterns)` - Multi-pattern subscription
- [ ] Write integration tests for all new methods
- [ ] Update documentation
- [ ] Release kryten-py v0.7.0

**Other Services** (kryten-llm, kryten-userstats, etc.):
- [ ] Add service discovery publishing on startup
- [ ] Add periodic heartbeat publishing (every 5-10 seconds)
- [ ] Listen to `kryten.lifecycle.robot.startup` for re-registration
- [ ] Listen to `kryten.service.discovery.poll` for re-registration
- [ ] Implement graceful shutdown on `kryten.lifecycle.group.restart` (if service matches)

### Phase 1: Core WebUI Infrastructure

- [ ] FastAPI backend scaffolding
  - [ ] Project structure (routers, services, models)
  - [ ] KrytenClient initialization and connection handling
  - [ ] WebSocket server for real-time updates
  - [ ] Error handling and logging
- [ ] Authentication system
  - [ ] OTP generation and PM sending
  - [ ] JWT token creation and validation
  - [ ] Redis session storage
  - [ ] Login/logout endpoints
  - [ ] Permission middleware (moderator/admin/owner)
- [ ] React frontend scaffolding
  - [ ] Vite + React + TypeScript setup
  - [ ] Routing (React Router)
  - [ ] Layout components (sidebar, header, content)
  - [ ] WebSocket client connection
  - [ ] State management (Zustand)
  - [ ] Theme system (dark/light mode)

### Phase 2: Service Discovery & Health Dashboard

- [ ] Backend: Service discovery integration
  - [ ] Subscribe to service discovery events
  - [ ] Subscribe to heartbeat events
  - [ ] Subscribe to lifecycle events
  - [ ] Maintain in-memory service registry
  - [ ] Heartbeat timeout detection
  - [ ] WebSocket broadcast of service updates
- [ ] Frontend: Service dashboard
  - [ ] Service cards component
  - [ ] Status indicators (healthy/degraded/failing/down)
  - [ ] Service grid layout
  - [ ] Service filtering and search
  - [ ] Service detail modal
  - [ ] Real-time status updates

### Phase 3: Event Stream Viewer

- [ ] Backend: Event streaming
  - [ ] Subscribe to all NATS events (wildcard)
  - [ ] Event filtering logic
  - [ ] Event buffering (circular buffer, last 1000)
  - [ ] WebSocket streaming to clients
- [ ] Frontend: Event viewer
  - [ ] Event stream component (virtual scrolling)
  - [ ] Event card design
  - [ ] Filter controls (subject, service, type, time)
  - [ ] Event detail modal (full JSON)
  - [ ] Event export (JSON, CSV)
  - [ ] Event statistics dashboard
  - [ ] Event search

### Phase 4: Log Aggregation

- [ ] Backend: Log aggregation
  - [ ] Subscribe to service log streams
  - [ ] Log buffering per service
  - [ ] Log level filtering
  - [ ] WebSocket log streaming
- [ ] Frontend: Log viewer
  - [ ] Unified log stream component
  - [ ] Service-specific log views
  - [ ] Log filtering (service, level, search)
  - [ ] Syntax highlighting
  - [ ] Log export
  - [ ] Error/warning quick filters

### Phase 5: Service Control Panel

- [ ] Backend: Service control
  - [ ] Restart service endpoint
  - [ ] Reload config endpoint
  - [ ] Shutdown service endpoint
  - [ ] Group restart endpoint
  - [ ] Permission checks (admin+ only)
  - [ ] Audit logging
- [ ] Frontend: Control UI
  - [ ] Service control buttons
  - [ ] Confirmation dialogs
  - [ ] Reason input fields
  - [ ] Action progress indicators
  - [ ] Audit log viewer

### Phase 6: Configuration Viewer

- [ ] Backend: Config access
  - [ ] Get kryten-robot config (existing method)
  - [ ] Get other service configs (future)
  - [ ] Config comparison logic
- [ ] Frontend: Config viewer
  - [ ] JSON viewer with syntax highlighting
  - [ ] Config browser (per service)
  - [ ] Config diff viewer
  - [ ] Config export

### Phase 7: User Statistics Integration

- [ ] Backend: kryten-userstats integration
  - [ ] Query user stats
  - [ ] Query channel stats
  - [ ] Subscribe to stats updates
- [ ] Frontend: User stats dashboard
  - [ ] Active users panel
  - [ ] User detail views
  - [ ] User statistics charts
  - [ ] Channel analytics

### Phase 8: Service Topology Visualization

- [ ] Backend: Topology data
  - [ ] Track service connections from event flow
  - [ ] Build service dependency graph
- [ ] Frontend: Topology view
  - [ ] Interactive service graph (D3.js or React Flow)
  - [ ] Connection visualization
  - [ ] Health heatmap
  - [ ] Drill-down to service details

### Phase 9: Polish & Advanced Features

- [ ] Analytics dashboard (system-wide metrics)
- [ ] Alert system (configurable thresholds)
- [ ] Notification system (browser notifications, email)
- [ ] Keyboard shortcuts
- [ ] Mobile responsiveness improvements
- [ ] Performance optimization (lazy loading, memoization)
- [ ] Export/reporting (PDF, CSV)
- [ ] Custom dashboard layouts (user preferences)
- [ ] Multi-channel support (monitor multiple CyTube channels)

## Security Considerations

### Authentication Security

- **OTP Codes**:
  - Cryptographically random (secrets.token_urlsafe)
  - Single-use only (delete after validation)
  - Time-limited (5 minutes default)
  - Rate-limited (max 3 attempts per 5 minutes per user)
- **JWT Tokens**:
  - Short-lived access tokens (1 hour)
  - Long-lived refresh tokens (7 days)
  - Secure signing key (256-bit minimum)
  - HttpOnly cookies for refresh tokens
  - Include user rank in token claims

### Authorization

- **Permission Checks**:
  - Verify user rank on every protected endpoint
  - Moderator: Read-only access
  - Admin: Service control access
  - Owner: Critical operations (shutdown robot)
- **Audit Logging**:
  - Log all service control actions
  - Include username, timestamp, reason, result
  - Store audit log in persistent storage

### Network Security

- **HTTPS/WSS**:
  - Enforce HTTPS in production
  - Use WSS for WebSocket connections
  - Valid SSL certificates (Let's Encrypt)
- **Input Validation**:
  - Validate all user inputs
  - Sanitize filter patterns (prevent injection)
  - Rate limiting on API endpoints
- **CORS**:
  - Strict CORS policy
  - Whitelist allowed origins

### NATS Security

- **Credentials**:
  - Store NATS credentials securely (file permissions 0600)
  - Use NATS authentication (user/pass or JWT)
  - Never expose credentials in UI or logs
- **Subject Permissions**:
  - WebUI service limited to necessary subjects
  - Read-only access to event streams
  - Write access only to control subjects (if admin+)

## Testing Strategy

### Unit Tests

- Authentication logic (OTP generation, JWT validation)
- Permission checks
- Event filtering logic
- Service registry management
- Utility functions

### Integration Tests

- KrytenClient connection and subscriptions
- Service discovery flow (mock services)
- Heartbeat timeout detection
- WebSocket message passing
- Service control actions
- Mock NATS for isolated testing

### E2E Tests

- Full authentication flow
- Service discovery and dashboard display
- Real-time event streaming
- Service control operations
- Log viewing
- Permission-based access control

### Manual Testing

- UI/UX validation
- Cross-browser testing (Chrome, Firefox, Safari)
- Performance under load (10+ services, 1000+ events/sec)
- Mobile responsiveness

## Deployment

### Production Requirements

- **Reverse Proxy**: nginx for SSL termination and load balancing
- **Process Manager**: systemd service for WebUI
- **Dependencies**:
  - NATS server (JetStream enabled)
  - Redis server (session storage)
  - kryten-robot (must be running)
- **Monitoring**: Prometheus + Grafana for WebUI itself
- **Logging**: Centralized logging (journald or syslog)
- **Backups**: Redis persistence, session data backup

### Environment Configuration

- **Development**: 
  - Local NATS and Redis
  - Mock services for testing
  - Hot reload enabled
- **Staging**: 
  - Full Kryten ecosystem
  - Test data
  - SSL certificates
- **Production**: 
  - Secured NATS connection
  - SSL/TLS enforced
  - Authentication required
  - Audit logging enabled

### Systemd Service

```ini
[Unit]
Description=Kryten WebUI
After=network.target nats.service redis.service kryten.service
Wants=nats.service redis.service kryten.service

[Service]
Type=simple
User=kryten
Group=kryten
WorkingDirectory=/opt/kryten/kryten-webui
Environment="PATH=/opt/kryten/kryten-webui/venv/bin:/usr/local/bin:/usr/bin:/bin"
ExecStart=/opt/kryten/kryten-webui/venv/bin/uvicorn kryten_webui.main:app --host 0.0.0.0 --port 8080
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

## Success Metrics

### Performance Goals

- Page load time < 2 seconds
- WebSocket latency < 100ms
- Event streaming throughput: 1000+ events/sec
- UI responsiveness < 50ms for interactions
- Support 50+ concurrent admin users

### User Experience Goals

- Intuitive navigation (minimal learning curve)
- Real-time updates feel instant
- No lag when scrolling logs/events
- Service discovery within 20 seconds of startup
- 99.9% uptime (same as NATS)

## Documentation

### User Documentation

- **Getting Started Guide**: Installation, configuration, first login
- **Feature Overview**: Dashboard tour, event viewer, log aggregation
- **Service Control Guide**: How to restart, reload config, shutdown services
- **Troubleshooting**: Common issues, FAQ
- **Keyboard Shortcuts Reference**

### Developer Documentation

- **Architecture Overview**: System design, component interaction
- **API Reference**: REST endpoints, WebSocket messages
- **Service Discovery Protocol**: How services register and send heartbeats
- **KrytenClient Integration**: Usage patterns, best practices
- **Contributing Guide**: Code style, testing, pull request process

## Architecture Principles

### kryten-py as the Single Source of Truth

1. **No Direct NATS Access**: WebUI backend MUST use kryten-py for all NATS communication
2. **Extend, Don't Bypass**: New functionality goes in kryten-py first
3. **Proper Abstraction**: kryten-py handles subject patterns, message formats, protocols
4. **Centralized Security**: Authentication and authorization in kryten-py
5. **Version Pinning**: WebUI requires specific kryten-py version

### Service Discovery Best Practices

1. **Registration on Startup**: Every service publishes to `kryten.service.discovery` immediately after NATS connection
2. **Periodic Heartbeats**: Every 5-10 seconds, publish to `kryten.service.heartbeat` with current status
3. **Health Status Honesty**: Report actual health (healthy/degraded/failing), not always "healthy"
4. **Graceful Shutdown**: Publish shutdown event before terminating
5. **Re-registration Support**: Listen to both `kryten.lifecycle.robot.startup` and `kryten.service.discovery.poll`

### UI/UX Best Practices

1. **Real-Time First**: Use WebSockets for all live data, not polling
2. **Optimistic Updates**: Show action result immediately, confirm with backend
3. **Progressive Disclosure**: Don't overwhelm users, show details on demand
4. **Consistent Status Colors**: Green=good, Yellow=warning, Red=error, Gray=down
5. **Performance Matters**: Virtual scrolling, lazy loading, efficient rendering

## Conclusion

The Kryten WebUI serves as a **mission control dashboard** for the entire Kryten microservice ecosystem. By providing real-time visibility into service health, event flow, logs, configuration, and user activity, it empowers developers and administrators to monitor, debug, and manage their Kryten deployment with confidence.

The architecture leverages existing lifecycle events while adding a robust service discovery and heartbeat system. All services register themselves on startup and send periodic heartbeats, allowing the WebUI to automatically discover and track the entire ecosystem without manual configuration.

The "snazzy" factor comes from real-time updates, smooth animations, intuitive design, and powerful features that make monitoring feel **alive** and engaging rather than static and boring. This is a tool you'll actually want to keep open on a second monitor.
