# 🚀 LiveKit Migration Guide - LEAP AI Vision V2

## Complete Migration Plan: WebSocket → LiveKit + Gemini Live API

---

## 📋 Table of Contents

1. [LiveKit Overview](#1-livekit-overview)
2. [Architecture Comparison](#2-architecture-comparison)
3. [Self-Hosting on Kubernetes](#3-self-hosting-on-kubernetes)
4. [Migration Requirements](#4-migration-requirements)
5. [Step-by-Step Migration Plan](#5-step-by-step-migration-plan)
6. [Code Changes Required](#6-code-changes-required)
7. [Testing & Validation](#7-testing--validation)
8. [Production Deployment](#8-production-deployment)
9. [Cost & Performance Analysis](#9-cost--performance-analysis)

---

## 1. LiveKit Overview

### What is LiveKit?

**LiveKit** is an open-source, WebRTC-based platform for real-time audio, video, and data communication. It provides:

- **Scalable Infrastructure**: Handle thousands of concurrent users
- **Low Latency**: Sub-100ms audio/video streaming
- **WebRTC Native**: Industry-standard real-time communication
- **Multi-Platform SDKs**: JavaScript, React, Python, Go, Swift, Kotlin
- **Cloud & Self-Hosted**: Deploy anywhere

### Why LiveKit for Your Use Case?

| Feature | Current (Pure WebSocket) | With LiveKit |
|---------|-------------------------|--------------|
| **Audio Quality** | Manual PCM handling | Optimized WebRTC codec (Opus) |
| **Video Streaming** | Base64 over WebSocket | Efficient WebRTC video tracks |
| **Scalability** | Single server instance | Load balanced, multi-node |
| **Network Resilience** | TCP only | UDP/TCP with ICE/TURN fallback |
| **Bandwidth** | ~512 kbps per user | Adaptive bitrate (50-300 kbps) |
| **Connection Recovery** | Manual reconnect | Automatic ICE restart |
| **Multi-Party** | Complex to implement | Built-in SFU architecture |

### LiveKit + Gemini Live API = Perfect Match

LiveKit provides the **transport layer** (WebRTC) while Gemini Live API provides the **AI intelligence**. This separation of concerns is ideal:

```
User (Browser/Mobile)
  ↓ WebRTC (LiveKit Client SDK)
LiveKit Server (Kubernetes)
  ↓ WebSocket/HTTP (Server SDK)
Your Backend (Python FastAPI)
  ↓ WebSocket (Gemini SDK)
Google Gemini Live API
```

---

## 2. Architecture Comparison

### Current Architecture (WebSocket-Only)

```mermaid
graph TB
    subgraph "Current System"
        CLIENT[Web Client]
        WS_SERVER[FastAPI WebSocket Server<br/>Port 8000]
        SESSION_MGR[Session Manager]
        GEMINI_CLIENT[Gemini Client]
        GEMINI_API[Google Gemini Live API]
        MONGO[(MongoDB)]
    end
    
    CLIENT -->|Raw WebSocket<br/>audio/video/text| WS_SERVER
    WS_SERVER --> SESSION_MGR
    SESSION_MGR --> GEMINI_CLIENT
    GEMINI_CLIENT -->|WebSocket| GEMINI_API
    SESSION_MGR --> MONGO
    
    style CLIENT fill:#e1f5fe
    style WS_SERVER fill:#fff3e0
    style GEMINI_API fill:#e8f5e8
```

**Limitations:**
- ❌ No WebRTC optimization (larger bandwidth)
- ❌ No adaptive bitrate
- ❌ TCP-only (higher latency, packet loss issues)
- ❌ Manual audio/video format handling
- ❌ Difficult to scale horizontally
- ❌ No built-in TURN/ICE server support

### Future Architecture (LiveKit + Gemini)

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web Browser]
        MOBILE[Mobile App]
    end
    
    subgraph "LiveKit Infrastructure (Kubernetes)"
        LK_LB[Load Balancer<br/>Ingress]
        LK_SFU1[LiveKit SFU Pod 1]
        LK_SFU2[LiveKit SFU Pod 2]
        LK_SFU3[LiveKit SFU Pod 3]
        REDIS[(Redis<br/>Pub/Sub)]
        TURN[TURN/ICE Server]
    end
    
    subgraph "Your Application Layer"
        AGENT_WORKER[LiveKit Agent Worker<br/>Python + Gemini Client]
        FASTAPI[FastAPI Backend<br/>Room Management + Tools]
        MONGO[(MongoDB<br/>Analytics)]
    end
    
    subgraph "External APIs"
        GEMINI_API[Google Gemini<br/>Live API]
        INDIAMART[IndiaMART API]
    end
    
    WEB -->|WebRTC| LK_LB
    MOBILE -->|WebRTC| LK_LB
    LK_LB --> LK_SFU1
    LK_LB --> LK_SFU2
    LK_LB --> LK_SFU3
    LK_SFU1 <--> REDIS
    LK_SFU2 <--> REDIS
    LK_SFU3 <--> REDIS
    LK_SFU1 --> TURN
    
    LK_SFU1 -->|Room Events| AGENT_WORKER
    AGENT_WORKER -->|WebSocket| GEMINI_API
    AGENT_WORKER --> FASTAPI
    FASTAPI --> INDIAMART
    FASTAPI --> MONGO
    
    style WEB fill:#e1f5fe
    style MOBILE fill:#e1f5fe
    style LK_SFU1 fill:#f3e5f5
    style LK_SFU2 fill:#f3e5f5
    style LK_SFU3 fill:#f3e5f5
    style AGENT_WORKER fill:#e8f5e8
    style GEMINI_API fill:#fff3e0
```

**Benefits:**
- ✅ WebRTC optimized transport (Opus audio, VP8/H.264 video)
- ✅ Adaptive bitrate based on network conditions
- ✅ UDP with TCP/TURN fallback
- ✅ Horizontal scaling (multiple SFU pods)
- ✅ Built-in ICE/TURN server support
- ✅ Multi-party support (future expansion)
- ✅ Better mobile network handling

---

## 3. Self-Hosting on Kubernetes

### 3.1 Infrastructure Requirements

#### Minimum Requirements (Development/Staging)

| Component | Resources | Quantity |
|-----------|-----------|----------|
| **LiveKit SFU** | 2 CPU, 4GB RAM | 1 pod |
| **Redis** | 1 CPU, 2GB RAM | 1 pod |
| **TURN Server** | 1 CPU, 2GB RAM | 1 pod (optional) |
| **Agent Workers** | 2 CPU, 4GB RAM | 1-2 pods |
| **Total** | ~8 CPU, 16GB RAM | 4-5 pods |

#### Production Requirements (High Availability)

| Component | Resources | Quantity | Notes |
|-----------|-----------|----------|-------|
| **LiveKit SFU** | 4 CPU, 8GB RAM | 3-5 pods | Auto-scale based on rooms |
| **Redis Cluster** | 2 CPU, 4GB RAM | 3 pods | Redis Sentinel/Cluster |
| **TURN Server** | 2 CPU, 4GB RAM | 2 pods | For restrictive networks |
| **Agent Workers** | 4 CPU, 8GB RAM | 3-5 pods | One per concurrent AI session |
| **Load Balancer** | Managed | 1 | AWS ELB / GKE LB |
| **Total** | ~40 CPU, 80GB RAM | 12-20 pods | |

#### Network Requirements

- **UDP Ports**: 20000-25000 (configurable)
- **TCP Ports**: 7880 (HTTP API), 7881 (WebRTC/TCP fallback)
- **Ingress**: HTTPS (443) for WebSocket signaling
- **Egress**: Port 443 for Gemini API access

### 3.2 Kubernetes Deployment Architecture

```yaml
# Namespace structure
apiVersion: v1
kind: Namespace
metadata:
  name: livekit-production
  labels:
    app: livekit
    environment: production
```

#### Required Kubernetes Resources

1. **ConfigMap** - LiveKit configuration
2. **Secret** - API keys (Gemini, LiveKit, IndiaMART)
3. **Deployment** - LiveKit SFU servers
4. **StatefulSet** - Redis cluster
5. **Service** - LoadBalancer for external access
6. **Ingress** - HTTPS termination
7. **HorizontalPodAutoscaler** - Auto-scaling based on load
8. **PersistentVolumeClaim** - Recording storage (if needed)

### 3.3 Step-by-Step Kubernetes Setup

#### Step 1: Create Namespace and ConfigMap

```bash
# Create namespace
kubectl create namespace livekit-production

# Create LiveKit ConfigMap
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: livekit-config
  namespace: livekit-production
data:
  livekit.yaml: |
    port: 7880
    rtc:
      tcp_port: 7881
      port_range_start: 20000
      port_range_end: 25000
      use_external_ip: true
      ice_servers:
        - urls:
          - stun:stun.l.google.com:19302
    redis:
      address: redis-service:6379
    keys:
      APIAbcDefGhi123: JklMnoPqrStuVwxYz456
    logging:
      level: info
      json: true
      sample: false
    turn:
      enabled: true
      domain: turn.yourdomain.com
      tls_port: 5349
      udp_port: 3478
EOF
```

#### Step 2: Deploy Redis

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: livekit-production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: livekit-production
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
EOF
```

#### Step 3: Deploy LiveKit SFU Using Helm

```bash
# Add LiveKit Helm repo
helm repo add livekit https://helm.livekit.io
helm repo update

# Create values.yaml for customization
cat > livekit-values.yaml <<EOF
replicaCount: 3

image:
  repository: livekit/livekit-server
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"

config:
  port: 7880
  rtc:
    tcp_port: 7881
    port_range_start: 20000
    port_range_end: 25000
    use_external_ip: true
  redis:
    address: "redis-service:6379"
  keys:
    APIAbcDefGhi123: JklMnoPqrStuVwxYz456

resources:
  requests:
    cpu: 2000m
    memory: 4Gi
  limits:
    cpu: 4000m
    memory: 8Gi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

nodeSelector:
  workload: media-processing

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - livekit
        topologyKey: kubernetes.io/hostname
EOF

# Deploy LiveKit
helm install livekit livekit/livekit-server \
  --namespace livekit-production \
  --values livekit-values.yaml \
  --wait
```

#### Step 4: Create Ingress for HTTPS

```bash
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: livekit-ingress
  namespace: livekit-production
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/websocket-services: "livekit"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  tls:
  - hosts:
    - livekit.yourdomain.com
    secretName: livekit-tls
  rules:
  - host: livekit.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: livekit
            port:
              number: 7880
EOF
```

#### Step 5: Deploy TURN Server (Optional - for restrictive networks)

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coturn
  namespace: livekit-production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coturn
  template:
    metadata:
      labels:
        app: coturn
    spec:
      containers:
      - name: coturn
        image: coturn/coturn:latest
        ports:
        - containerPort: 3478
          protocol: UDP
        - containerPort: 3478
          protocol: TCP
        - containerPort: 5349
          protocol: TCP
        env:
        - name: REALM
          value: "yourdomain.com"
        - name: STATIC_AUTH_SECRET
          valueFrom:
            secretKeyRef:
              name: turn-secret
              key: auth-secret
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1000m"
            memory: "2Gi"
---
apiVersion: v1
kind: Service
metadata:
  name: turn-service
  namespace: livekit-production
spec:
  type: LoadBalancer
  selector:
    app: coturn
  ports:
  - name: turn-udp
    port: 3478
    targetPort: 3478
    protocol: UDP
  - name: turn-tcp
    port: 3478
    targetPort: 3478
    protocol: TCP
  - name: turns
    port: 5349
    targetPort: 5349
    protocol: TCP
EOF
```

### 3.4 Verification

```bash
# Check all pods are running
kubectl get pods -n livekit-production

# Check services
kubectl get svc -n livekit-production

# Check logs
kubectl logs -n livekit-production -l app=livekit --tail=100

# Test LiveKit API health
curl https://livekit.yourdomain.com/healthz
```

---

## 4. Migration Requirements

### 4.1 Infrastructure Components Needed

| Component | Purpose | Status |
|-----------|---------|--------|
| **LiveKit Server** | WebRTC SFU for media routing | ✅ Deploy on K8s |
| **Redis** | Pub/sub for multi-node coordination | ✅ Deploy on K8s |
| **TURN Server** | NAT traversal for restricted networks | ⚠️ Optional (use Google STUN first) |
| **LiveKit Agents** | Python workers for Gemini integration | 🔨 New component to build |
| **Token Generation Service** | Generate JWT tokens for room access | 🔨 Add to FastAPI backend |

### 4.2 Software Dependencies

#### Backend (Python)

```bash
# New dependencies to add
pip install livekit-server-sdk>=0.10.0     # LiveKit Server SDK
pip install livekit-agents>=0.8.0          # LiveKit Agents framework
pip install livekit-plugins-google         # Google STT/TTS plugins
pip install PyJWT>=2.8.0                   # JWT token generation
```

#### Frontend (React)

```bash
# New dependencies to add
npm install @livekit/components-react@^2.0.0
npm install @livekit/components-core@^0.11.0
npm install livekit-client@^2.0.0
```

### 4.3 Code Changes Overview

| File | Current Lines | Changes Required | Effort |
|------|---------------|------------------|--------|
| `api/core/session_manager.py` | 1055 | Major refactor - Replace WebSocket with LiveKit Room | 🔴 High |
| `api/core/gemini_client.py` | 950 | Moderate - Adapt for LiveKit Agent | 🟡 Medium |
| `api/routes/websocket.py` | 548 | Replace with LiveKit room endpoint | 🔴 High |
| `api/main.py` | 352 | Add LiveKit SDK initialization | 🟢 Low |
| `frontend-v2/src/services/WebSocketService.js` | 362 | Replace with LiveKit Client | 🔴 High |
| `frontend-v2/src/components/LiveView.js` | 1720 | Replace media capture with LiveKit hooks | 🟡 Medium |
| **New Files** | - | LiveKit Agent worker (new) | 🔴 High |

---

## 5. Step-by-Step Migration Plan

### Phase 1: Infrastructure Setup (Week 1)

**Goal:** Deploy LiveKit infrastructure on Kubernetes

- [ ] **Day 1-2:** Deploy LiveKit SFU on Kubernetes
  - Create namespace and ConfigMap
  - Deploy Redis
  - Deploy LiveKit using Helm
  - Configure Ingress and TLS

- [ ] **Day 3:** Deploy TURN server (if needed)
  - Test with restrictive network scenarios
  - Configure fallback mechanisms

- [ ] **Day 4-5:** Monitoring and validation
  - Deploy Prometheus + Grafana for metrics
  - Configure alerts
  - Load testing with simple WebRTC clients

**Validation Criteria:**
- ✅ LiveKit health endpoint returns 200
- ✅ Can create/join rooms via LiveKit API
- ✅ WebRTC media flows through SFU
- ✅ Auto-scaling triggers correctly

### Phase 2: Backend Migration (Week 2-3)

**Goal:** Create LiveKit Agent workers and migrate backend logic

#### 2.1 Create LiveKit Agent Worker

Create new file: `leapAi/visionv2/api/agents/gemini_agent.py`

```python
"""
LiveKit Agent Worker for Gemini Live API Integration
"""
import logging
from livekit import rtc
from livekit.agents import JobContext, WorkerOptions, cli
from livekit.plugins import google
import asyncio

# Import your existing Gemini client (adapted)
from ..core.gemini_client import GeminiLiveClient
from ..tools import get_tool_manager

logger = logging.getLogger(__name__)


class GeminiLiveAgent:
    """Agent that connects LiveKit room to Gemini Live API"""
    
    def __init__(self, ctx: JobContext):
        self.ctx = ctx
        self.room = ctx.room
        self.gemini_client = None
        self.tool_manager = get_tool_manager()
        
    async def start(self):
        """Start the agent when participant joins"""
        logger.info(f"🚀 Agent starting for room: {self.room.name}")
        
        # Wait for participant to join
        participant = await self.room.wait_for_participant()
        logger.info(f"👤 Participant joined: {participant.identity}")
        
        # Subscribe to participant's audio track
        audio_track = None
        for publication in participant.track_publications.values():
            if publication.track and publication.track.kind == rtc.TrackKind.KIND_AUDIO:
                audio_track = publication.track
                break
        
        if not audio_track:
            logger.error("No audio track found from participant")
            return
        
        # Initialize Gemini client
        self.gemini_client = GeminiLiveClient(video_mode="audio_only")
        await self.gemini_client.connect()
        
        # Create audio pipeline
        # Input: participant -> Gemini
        asyncio.create_task(self._process_input_audio(audio_track))
        
        # Output: Gemini -> participant
        asyncio.create_task(self._process_output_audio())
        
        logger.info("✅ Agent pipeline established")
    
    async def _process_input_audio(self, audio_track):
        """Stream audio from participant to Gemini"""
        audio_stream = rtc.AudioStream(audio_track)
        
        async for frame in audio_stream:
            # Convert LiveKit audio frame to PCM bytes
            audio_data = frame.data.tobytes()
            
            # Send to Gemini (your existing method)
            await self.gemini_client.send_audio(audio_data)
    
    async def _process_output_audio(self):
        """Stream audio from Gemini to participant"""
        # Create audio source for publishing
        source = rtc.AudioSource(24000, 1)  # 24kHz, mono
        track = rtc.LocalAudioTrack.create_audio_track("agent-voice", source)
        
        # Publish track to room
        await self.room.local_participant.publish_track(track)
        
        # Listen for Gemini responses
        async for response in self.gemini_client.listen_for_responses():
            if hasattr(response, 'data') and response.data:
                # Convert Gemini audio to LiveKit format
                audio_frame = rtc.AudioFrame(
                    data=response.data,
                    sample_rate=24000,
                    num_channels=1,
                    samples_per_channel=len(response.data) // 2
                )
                await source.capture_frame(audio_frame)


async def entrypoint(ctx: JobContext):
    """Entry point for LiveKit agent"""
    agent = GeminiLiveAgent(ctx)
    await agent.start()


if __name__ == "__main__":
    # Run the agent worker
    cli.run_app(
        WorkerOptions(
            entrypoint_fnc=entrypoint,
            agent_name="gemini-live-agent",
        )
    )
```

#### 2.2 Update FastAPI Backend for Room Management

Update `leapAi/visionv2/api/routes/livekit_rooms.py` (new file):

```python
"""
LiveKit Room Management Routes
"""
from fastapi import APIRouter, HTTPException, Depends
from livekit import api
import time
import os

router = APIRouter(prefix="/livekit", tags=["LiveKit"])

# Initialize LiveKit API client
livekit_api = api.LiveKitAPI(
    url=os.getenv("LIVEKIT_URL", "https://livekit.yourdomain.com"),
    api_key=os.getenv("LIVEKIT_API_KEY"),
    api_secret=os.getenv("LIVEKIT_API_SECRET"),
)


@router.post("/token")
async def create_token(
    room_name: str,
    participant_identity: str,
    participant_name: str = None
):
    """
    Generate LiveKit access token for a participant
    
    Replaces: /ws/stream/{session_id} endpoint
    """
    try:
        # Create access token
        token = api.AccessToken(
            api_key=os.getenv("LIVEKIT_API_KEY"),
            api_secret=os.getenv("LIVEKIT_API_SECRET"),
        )
        
        # Set token metadata
        token.with_identity(participant_identity)
        token.with_name(participant_name or participant_identity)
        token.with_grants(api.VideoGrants(
            room_join=True,
            room=room_name,
            can_publish=True,
            can_subscribe=True,
        ))
        
        # Set expiration (1 hour)
        token.with_ttl(3600)
        
        jwt_token = token.to_jwt()
        
        return {
            "token": jwt_token,
            "url": os.getenv("LIVEKIT_URL"),
            "room_name": room_name
        }
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.get("/rooms")
async def list_rooms():
    """List all active rooms"""
    try:
        rooms = await livekit_api.room.list_rooms()
        return {"rooms": [room.dict() for room in rooms]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.delete("/room/{room_name}")
async def delete_room(room_name: str):
    """Delete a room"""
    try:
        await livekit_api.room.delete_room(room_name)
        return {"message": f"Room {room_name} deleted"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Phase 3: Frontend Migration (Week 3-4)

**Goal:** Replace WebSocket client with LiveKit React components

#### 3.1 Create LiveKit Service

Create `leapAi/frontend-v2/src/services/LiveKitService.js`:

```javascript
import { Room, RoomEvent } from 'livekit-client';

class LiveKitService {
  constructor() {
    this.room = null;
    this.localParticipant = null;
    this.audioTrack = null;
    this.videoTrack = null;
  }

  /**
   * Connect to LiveKit room
   * Replaces: WebSocketService.connect()
   */
  async connect(roomName, participantName, onMessageCallback) {
    try {
      // Get token from backend
      const response = await fetch('/livekit/token', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          room_name: roomName,
          participant_identity: participantName,
          participant_name: participantName
        })
      });
      
      const { token, url } = await response.json();
      
      // Create room instance
      this.room = new Room({
        adaptiveStream: true,
        dynacast: true,
        videoCaptureDefaults: {
          resolution: { width: 640, height: 480 },
        },
      });
      
      // Set up event listeners
      this.room
        .on(RoomEvent.TrackSubscribed, this.handleTrackSubscribed.bind(this))
        .on(RoomEvent.TrackUnsubscribed, () => console.log('Track unsubscribed'))
        .on(RoomEvent.ActiveSpeakersChanged, () => console.log('Speakers changed'))
        .on(RoomEvent.Disconnected, () => console.log('Disconnected'))
        .on(RoomEvent.DataReceived, (payload, participant) => {
          // Handle data messages (like tool responses)
          const message = JSON.parse(new TextDecoder().decode(payload));
          if (onMessageCallback) {
            onMessageCallback(message);
          }
        });
      
      // Connect to room
      await this.room.connect(url, token);
      
      this.localParticipant = this.room.localParticipant;
      
      console.log('✅ Connected to LiveKit room:', roomName);
      
      return true;
    } catch (error) {
      console.error('❌ Failed to connect to LiveKit:', error);
      throw error;
    }
  }

  /**
   * Handle track subscriptions (agent audio output)
   */
  handleTrackSubscribed(track, publication, participant) {
    console.log('🎵 Track subscribed:', track.kind);
    
    if (track.kind === 'audio') {
      // This is the AI agent's voice
      const audioElement = track.attach();
      document.body.appendChild(audioElement);
      audioElement.play();
    }
  }

  /**
   * Publish microphone audio
   * Replaces: WebSocketService.sendAudio()
   */
  async enableMicrophone() {
    try {
      await this.room.localParticipant.setMicrophoneEnabled(true);
      console.log('🎤 Microphone enabled');
    } catch (error) {
      console.error('❌ Failed to enable microphone:', error);
      throw error;
    }
  }

  /**
   * Disable microphone
   */
  async disableMicrophone() {
    await this.room.localParticipant.setMicrophoneEnabled(false);
    console.log('🎤 Microphone disabled');
  }

  /**
   * Publish camera video
   * Replaces: WebSocketService.sendVideo()
   */
  async enableCamera() {
    try {
      await this.room.localParticipant.setCameraEnabled(true);
      console.log('📹 Camera enabled');
    } catch (error) {
      console.error('❌ Failed to enable camera:', error);
      throw error;
    }
  }

  /**
   * Disable camera
   */
  async disableCamera() {
    await this.room.localParticipant.setCameraEnabled(false);
    console.log('📹 Camera disabled');
  }

  /**
   * Send text message (tool call trigger)
   * Replaces: WebSocketService.sendText()
   */
  async sendText(text) {
    const message = JSON.stringify({
      type: 'text_message',
      text: text,
      timestamp: new Date().toISOString()
    });
    
    const encoder = new TextEncoder();
    await this.room.localParticipant.publishData(encoder.encode(message));
  }

  /**
   * Disconnect from room
   * Replaces: WebSocketService.disconnect()
   */
  async disconnect() {
    if (this.room) {
      await this.room.disconnect();
      this.room = null;
      console.log('🔌 Disconnected from LiveKit');
    }
  }

  /**
   * Get connection status
   */
  getConnectionStatus() {
    return {
      isConnected: this.room && this.room.state === 'connected',
      roomName: this.room?.name,
      participantCount: this.room?.participants.size
    };
  }
}

export default new LiveKitService();
```

#### 3.2 Update LiveView Component

Update `leapAi/frontend-v2/src/components/LiveView.js`:

```javascript
// Replace WebSocketService import
// import WebSocketService from '../services/WebSocketService';
import LiveKitService from '../services/LiveKitService';

// In LiveView component:

const handleConnect = useCallback(async () => {
  try {
    const roomName = `room-${sessionId}`;
    const participantName = mobile || `user-${Date.now()}`;
    
    await LiveKitService.connect(roomName, participantName, handleLiveKitMessage);
    
    setIsConnected(true);
    console.log('✅ Connected to LiveKit');
  } catch (error) {
    console.error('❌ Connection failed:', error);
  }
}, [sessionId, mobile]);

const handleLiveKitMessage = useCallback((message) => {
  // Same as handleWebSocketMessage
  handleWebSocketMessage(message);
}, [handleWebSocketMessage]);

const toggleMicrophone = useCallback(async () => {
  if (isMicOn) {
    await LiveKitService.disableMicrophone();
  } else {
    await LiveKitService.enableMicrophone();
  }
  setIsMicOn(!isMicOn);
}, [isMicOn]);

const toggleCamera = useCallback(async () => {
  if (isCameraOn) {
    await LiveKitService.disableCamera();
  } else {
    await LiveKitService.enableCamera();
  }
  setIsCameraOn(!isCameraOn);
}, [isCameraOn]);
```

### Phase 4: Testing & Validation (Week 4-5)

**Goal:** Comprehensive testing of migrated system

- [ ] **Unit Tests**
  - Agent worker logic
  - Token generation
  - Room management API

- [ ] **Integration Tests**
  - End-to-end audio flow
  - Video streaming
  - Tool execution
  - Session analytics

- [ ] **Load Tests**
  - 100 concurrent users
  - Network quality variations
  - Auto-scaling behavior

- [ ] **User Acceptance Testing**
  - Real user scenarios
  - Mobile testing (iOS/Android)
  - Different network conditions

### Phase 5: Production Deployment (Week 5-6)

**Goal:** Gradual rollout with monitoring

- [ ] **Canary Deployment**
  - Route 10% traffic to LiveKit
  - Monitor metrics
  - Compare with WebSocket performance

- [ ] **Gradual Rollout**
  - 25% traffic
  - 50% traffic
  - 100% traffic

- [ ] **Monitoring**
  - Set up Grafana dashboards
  - Configure alerts
  - Monitor costs

---

## 6. Code Changes Required

### 6.1 Backend Changes Summary

| File | Action | Description |
|------|--------|-------------|
| `api/agents/gemini_agent.py` | **NEW** | LiveKit Agent worker |
| `api/routes/livekit_rooms.py` | **NEW** | Room management API |
| `api/core/session_manager.py` | **MODIFY** | Remove WebSocket, add LiveKit room callbacks |
| `api/core/gemini_client.py` | **MODIFY** | Adapt for LiveKit audio frames |
| `api/routes/websocket.py` | **DEPRECATE** | Keep for backward compatibility initially |
| `config.py` | **MODIFY** | Add LiveKit configuration |

### 6.2 Frontend Changes Summary

| File | Action | Description |
|------|--------|-------------|
| `src/services/LiveKitService.js` | **NEW** | LiveKit client wrapper |
| `src/services/WebSocketService.js` | **DEPRECATE** | Keep for backward compatibility |
| `src/components/LiveView.js` | **MODIFY** | Use LiveKit hooks |
| `src/hooks/useWebSocket.js` | **REPLACE** | Create `useLiveKit.js` |
| `package.json` | **MODIFY** | Add LiveKit dependencies |

---

## 7. Testing & Validation

### 7.1 Testing Checklist

#### Infrastructure Tests
- [ ] LiveKit pods are running and healthy
- [ ] Redis connection is stable
- [ ] Load balancer distributes traffic correctly
- [ ] Auto-scaling triggers at 70% CPU
- [ ] TURN server works in restricted networks

#### Functional Tests
- [ ] Can create and join rooms
- [ ] Audio flows bidirectionally
- [ ] Video streaming works
- [ ] Text messages are delivered
- [ ] Tool execution works (supplier search)
- [ ] Session analytics are logged

#### Performance Tests
- [ ] Audio latency < 200ms
- [ ] Video latency < 500ms
- [ ] Can handle 100 concurrent users
- [ ] Bandwidth usage is optimized
- [ ] Memory usage is stable

#### Edge Cases
- [ ] Network disconnection recovery
- [ ] Agent crash recovery
- [ ] Redis failure handling
- [ ] Rate limiting works

### 7.2 Load Testing Script

```python
# load_test.py
import asyncio
from livekit import api
import aiohttp

async def simulate_user(user_id: int):
    """Simulate a single user session"""
    # Get token
    async with aiohttp.ClientSession() as session:
        async with session.post(
            'https://your-api.com/livekit/token',
            json={
                'room_name': f'test-room-{user_id % 10}',
                'participant_identity': f'user-{user_id}'
            }
        ) as resp:
            data = await resp.json()
    
    # Connect to room
    room = api.Room()
    await room.connect(data['url'], data['token'])
    
    # Simulate 2-minute conversation
    await asyncio.sleep(120)
    
    # Disconnect
    await room.disconnect()

async def main():
    # Simulate 100 concurrent users
    tasks = [simulate_user(i) for i in range(100)]
    await asyncio.gather(*tasks)

asyncio.run(main())
```

---

## 8. Production Deployment

### 8.1 Deployment Checklist

- [ ] **DNS Configuration**
  - `livekit.yourdomain.com` → LoadBalancer IP
  - `turn.yourdomain.com` → TURN server IP

- [ ] **SSL Certificates**
  - Install Let's Encrypt certificates
  - Configure cert-manager for auto-renewal

- [ ] **Secrets Management**
  - Store API keys in Kubernetes Secrets
  - Rotate secrets regularly

- [ ] **Monitoring**
  - Deploy Prometheus + Grafana
  - Configure alerts (Slack/PagerDuty)

- [ ] **Logging**
  - ELK stack or Loki for log aggregation
  - Structured JSON logs

- [ ] **Backup**
  - Redis snapshots
  - MongoDB backups
  - Recording storage (if enabled)

### 8.2 Rollback Plan

If migration fails:

1. **Immediate Rollback**
   ```bash
   # Route all traffic back to WebSocket
   kubectl patch ingress main-ingress -p '{"spec":{"rules":[{"http":{"paths":[{"path":"/ws","backend":{"serviceName":"websocket-service"}}]}}]}}'
   ```

2. **Scale down LiveKit**
   ```bash
   kubectl scale deployment livekit --replicas=0 -n livekit-production
   ```

3. **Notify users** via in-app message

---

## 9. Cost & Performance Analysis

### 9.1 Cost Comparison

#### Current WebSocket System

| Resource | Quantity | Cost/Month |
|----------|----------|-----------|
| EC2/GKE Nodes | 3 × 4vCPU, 8GB | $300 |
| Load Balancer | 1 | $20 |
| MongoDB | 1 cluster | $150 |
| Total | - | **$470/month** |

#### With LiveKit

| Resource | Quantity | Cost/Month |
|----------|----------|-----------|
| LiveKit SFU Pods | 3-5 × 4vCPU, 8GB | $400 |
| Redis | 1 cluster | $50 |
| TURN Server | 2 × 2vCPU, 4GB | $100 |
| Agent Workers | 3-5 × 4vCPU, 8GB | $400 |
| Load Balancer | 1 | $20 |
| MongoDB | 1 cluster | $150 |
| Total | - | **$1,120/month** |

**Cost Increase:** ~138% ($650 more per month)

**Benefits for the cost:**
- Better audio quality (Opus codec)
- Lower bandwidth usage per user
- Better scalability (can handle 5x more users)
- Mobile network optimization
- Multi-party support (future)

### 9.2 Performance Comparison

| Metric | Current | With LiveKit | Improvement |
|--------|---------|--------------|-------------|
| Audio Latency | 200-500ms | 100-200ms | 2x better |
| Video Latency | 500-1000ms | 200-400ms | 2-3x better |
| Bandwidth/User | 512 kbps | 64-256 kbps | 2-8x better |
| Connection Success | 95% | 98% | 3% better |
| Mobile Quality | Poor | Good | Major improvement |
| Max Concurrent Users | 100 | 500+ | 5x better |

---

## 10. Timeline Summary

| Phase | Duration | Effort | Risk |
|-------|----------|--------|------|
| Infrastructure Setup | 1 week | 40 hours | Low |
| Backend Migration | 2 weeks | 80 hours | Medium |
| Frontend Migration | 1-2 weeks | 60 hours | Medium |
| Testing & Validation | 1-2 weeks | 60 hours | Low |
| Production Deployment | 1 week | 20 hours | High |
| **Total** | **5-7 weeks** | **260 hours** | **Medium** |

---

## 11. Decision Matrix

### Should You Migrate?

| Factor | Weight | Score (1-10) | Weighted |
|--------|--------|--------------|----------|
| **Scalability Needs** | 25% | 9 | 2.25 |
| **Audio Quality Requirements** | 20% | 8 | 1.60 |
| **Mobile Support** | 15% | 9 | 1.35 |
| **Development Effort** | 20% | 4 | 0.80 |
| **Cost Impact** | 20% | 5 | 1.00 |
| **Total Score** | 100% | - | **7.0/10** |

**Recommendation:** ✅ **Proceed with migration**

**Reasoning:**
- Strong benefits in scalability and quality
- Medium development effort (5-7 weeks)
- Cost increase is justified by 5x capacity improvement
- Critical for mobile user experience

---

## 12. Next Steps

### Immediate Actions (This Week)

1. **Provision Kubernetes cluster** for LiveKit staging environment
2. **Deploy LiveKit** using Helm on staging
3. **Create proof-of-concept** with simple audio agent
4. **Test** basic audio flow end-to-end

### Questions to Answer

1. Do you want to keep WebSocket as fallback initially?
2. What's your target date for production launch?
3. Do you need multi-party support in future?
4. What's your expected user growth?
5. Do you want recording/playback features?

---

## 13. Resources

- **LiveKit Documentation:** https://docs.livekit.io
- **LiveKit Agents:** https://docs.livekit.io/agents/
- **Kubernetes Guide:** https://docs.livekit.io/deploy/kubernetes/
- **Helm Charts:** https://github.com/livekit/charts
- **Example Projects:** https://github.com/livekit/agents/tree/main/examples

---

## 📞 Support

For questions or assistance with the migration:
- Review this guide thoroughly
- Test in staging environment first
- Document any issues encountered
- Reach out to LiveKit community for platform-specific questions

**Good luck with your migration! 🚀**

