# Hermes Agent Stability Guide

## Overview

This document outlines stability requirements, monitoring procedures, and operational runbooks for the Hermes agent runtime.

## Architecture Stability

### Core Components

| Component | Stability Requirement | Monitoring |
|-----------|----------------------|------------|
| **CLI** (`cli.py`) | Always available | Startup health check |
| **Agent Runtime** (`agent/`) | Crash-resistant | Process monitor |
| **Gateway** (`gateway/`) | High availability | Uptime monitor |
| **Skills Loader** (`skills/`) | Fail-safe | Load failure alert |
| **MCP Server** (`mcp_serve.py`) | Low latency | Response time monitor |
| **TUI Gateway** (`tui_gateway/`) | Responsive | User experience monitor |

### Process Management

#### Systemd Service (Linux)
```ini
[Unit]
Description=Hermes Agent
After=network.target

[Service]
Type=simple
User=hermes
WorkingDirectory=/opt/hermes
ExecStart=/opt/hermes/venv/bin/python cli.py
Restart=always
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

#### Windows Service
```powershell
# Install as Windows service
nssm install HermesAgent "C:\hermes\venv\Scripts\python.exe" "C:\hermes\cli.py"
nssm set HermesAgent AppDirectory "C:\hermes"
nssm set HermesAgent RestartService 1
```

## Health Checks

### Pre-Flight Validation
```bash
# Validate environment
python -c "import hermes; print('OK')"

# Check config
python cli.py --validate-config

# Verify skills
python cli.py --list-skills

# Test gateway
curl -f http://localhost:8080/health
```

### Runtime Monitoring
```bash
# Agent status
python cli.py --status

# Memory usage
python cli.py --memory-usage

# Active sessions
python cli.py --sessions

# Skill health
python cli.py --skill-status
```

## Stability Metrics

### Key Performance Indicators
| Metric | Target | Alert Threshold |
|--------|--------|-----------------|
| Uptime | 99.9% | < 99.5% |
| Response time | < 500ms | > 1000ms |
| Memory usage | < 2GB | > 3GB |
| CPU usage | < 50% | > 80% |
| Error rate | < 0.1% | > 1% |
| Session success | > 99% | < 95% |

### Monitoring Script
```bash
#!/bin/bash
# monitor-hermes.sh

# Check if agent is running
if ! pgrep -f "cli.py" > /dev/null; then
    echo "ALERT: Hermes agent is not running"
    systemctl restart hermes-agent
    exit 1
fi

# Check memory usage
MEMORY=$(ps aux | grep cli.py | awk '{print $6}' | head -1)
if [ $MEMORY -gt 3000000 ]; then
    echo "WARNING: High memory usage: ${MEMORY}KB"
fi

# Check response time
START=$(date +%s%N)
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/health
END=$(date +%s%N)
ELAPSED=$((($END - $START) / 1000000))
if [ $ELAPSED -gt 1000 ]; then
    echo "WARNING: Slow response time: ${ELAPSED}ms"
fi
```

## Error Handling

### Graceful Degradation
1. **Skill Failure**: Disable skill, continue with remaining skills
2. **LLM API Failure**: Queue request, retry with exponential backoff
3. **Memory Corruption**: Switch to read-only mode, alert admin
4. **Gateway Timeout**: Retry 3x, then failover to backup

### Retry Logic
```python
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10),
    retry=retry_if_exception_type(APIError)
)
def call_llm_with_retry(prompt):
    return llm_client.complete(prompt)
```

### Circuit Breaker
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failures = 0
        self.last_failure = None
    
    def call(self, func, *args, **kwargs):
        if self.is_open():
            raise CircuitBreakerOpen("Circuit breaker is open")
        try:
            result = func(*args, **kwargs)
            self.reset()
            return result
        except Exception as e:
            self.record_failure()
            raise
```

## Logging & Observability

### Log Levels
- **ERROR**: System failures, requires immediate action
- **WARNING**: Degraded performance, investigate soon
- **INFO**: Normal operations, audit trail
- **DEBUG**: Detailed diagnostics, development only

### Log Format
```json
{
  "timestamp": "2026-07-09T11:00:00Z",
  "level": "INFO",
  "component": "agent",
  "message": "Session started",
  "session_id": "abc123",
  "user": "zqmco",
  "duration_ms": 150
}
```

### Key Logs to Monitor
- `agent.log` - Main agent operations
- `gateway.log` - API gateway requests
- `skills.log` - Skill execution
- `memory.log` - Memory operations
- `error.log` - All errors

## Recovery Procedures

### Agent Crash
1. **Automatic**: Systemd/NSSM restarts within 10s
2. **Validation**: Run pre-flight checks
3. **State Recovery**: Restore from last checkpoint
4. **Notification**: Alert if crash > 3x in 1 hour

### Data Corruption
1. **Stop agent**: Prevent further corruption
2. **Assess damage**: Check logs for extent
3. **Restore**: From last known good backup
4. **Validate**: Run integrity checks
5. **Resume**: Start in read-only mode first

### Performance Degradation
1. **Identify bottleneck**: CPU, memory, I/O, network
2. **Scale resources**: Add CPU/memory if needed
3. **Optimize**: Profile hot paths
4. **Cache**: Add caching for frequent operations
5. **Queue**: Implement request queuing

## Testing Strategy

### Unit Tests
```bash
pytest tests/unit/ -v --cov=agent --cov=skills
```

### Integration Tests
```bash
pytest tests/integration/ -v --tb=short
```

### Load Tests
```bash
# Simulate 100 concurrent sessions
python tests/load_test.py --concurrent 100 --duration 60s
```

### Chaos Engineering
```bash
# Kill agent randomly, verify recovery
python tests/chaos.py --kill-probability 0.1 --duration 300s
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests pass
- [ ] Config validated
- [ ] Skills tested
- [ ] Backups current
- [ ] Monitoring active
- [ ] Rollback plan ready

### Deployment
- [ ] Deploy to staging
- [ ] Run smoke tests
- [ ] Monitor for 1 hour
- [ ] Deploy to production
- [ ] Monitor for 24 hours

### Post-Deployment
- [ ] Verify metrics
- [ ] Check logs for errors
- [ ] Confirm skills loaded
- [ ] Validate sessions working
- [ ] Update runbook if needed

## Emergency Contacts

| Role | Contact | Escalation |
|------|---------|------------|
| **On-Call Engineer** | @hermes-oncall | 15 min |
| **Senior Engineer** | @hermes-senior | 30 min |
| **Admin** | security@zqmcomputing.com | 1 hour |
| **Emergency** | +1-XXX-XXX-XXXX | Immediate |

## Related Documents

- [hermes-config/STABILITY.md](../hermes-config/STABILITY.md) - Config stability
- [SECURITY.md](../SECURITY.md) - Security policy
- [CONTRIBUTING.md](../CONTRIBUTING.md) - Development guide