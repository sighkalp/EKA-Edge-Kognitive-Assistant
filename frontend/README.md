# Frontend Dashboard

**Responsibility**: Display mission state, experiment progress, alerts, and facilitate astronaut interaction.

---

## Overview

The Frontend Dashboard is the astronaut's and mission control's window into the mission. It displays:

- Real-time video with detected objects
- Current activity and procedure step
- Active alerts and warnings
- Q&A interface
- Mission timeline
- System status

**Owner**: Member 6  
**Framework**: React  
**Real-Time**: WebSocket  
**Styling**: TailwindCSS  

---

## Features

### Astronaut View
- Video feed with object bounding boxes
- Current procedure step and progress
- Prominent alert display
- Q&A input and response
- Emergency stop button

### Mission Control View
- Overview of active missions
- Experiment state for multiple astronauts
- Complete event timeline
- Audit log viewer
- Analytics and reporting

---

## Installation

```bash
cd frontend
npm install
npm start
```

Runs at `http://localhost:3000`

---

## Structure

```
frontend/
├── src/
│   ├── components/
│   │   ├── VideoFeed.jsx
│   │   ├── ProcedurePanel.jsx
│   │   ├── AlertBoard.jsx
│   │   ├── QAInterface.jsx
│   │   └── Timeline.jsx
│   ├── pages/
│   │   ├── AstronautView.jsx
│   │   ├── MissionControl.jsx
│   │   └── Dashboard.jsx
│   ├── services/
│   │   ├── api.js
│   │   └── websocket.js
│   └── App.jsx
├── public/
└── package.json
```

---

## Real-Time Updates

Uses WebSocket to receive updates from backend:

```javascript
const ws = new WebSocket('ws://localhost:8000/ws/mission/mission_001');

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  // Handle: vision, activity, experiment state, safety alerts
  updateDashboard(update);
};
```

---

## Testing

```bash
npm test

# With coverage
npm test -- --coverage
```

---

## Build

```bash
npm run build
# Outputs to build/
```

---

## Deployment

```bash
npm run build
npm run serve
# Or use nginx to serve build/ folder
```

---

**For detailed setup**: See `frontend/README.md` after implementation

**Status**: Ready for implementation
