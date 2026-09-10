


```bash
#!/usr/bin/env bash
#
# vlan-healthcheck.sh
#
# Daily health check for the Home SOC Lab's VLAN segmentation.
# Run from a host in the Management VLAN (10.10.10.0/24) with:
#   - Network reachability to all VLAN gateways
#   - SSH key-based access to pfSense
#
# What it checks:
#   1. Each VLAN gateway responds (pfSense interface is up)
#   2. Allowed cross-VLAN paths still work (e.g. Target -> SOC on Wazuh port)
#   3. Suricata inline instances are running on pfSense
#   4. Attacker VLAN isolation is still enforced (should NOT reach Management/SOC)
#
# Exit codes: 0 = all checks passed, 1 = one or more checks failed
#
# Suggested cron entry (run daily at 06:00, log to file):
#   0 6 * * * /path/to/vlan-healthcheck.sh >> /var/log/vlan-healthcheck.log 2>&1

set -uo pipefail

# ---------------------------------------------------------------------------
# CONFIG — edit these to match your lab
# ---------------------------------------------------------------------------
declare -A VLAN_GATEWAYS=(
  [MGMT]="10.10.10.1"
  [SOC]="10.10.20.1"
  [TARGET]="10.10.30.1"
  [ATTACKER]="10.10.40.1"
)

PFSENSE_HOST="10.10.10.1"      # pfSense management IP (adjust if OOB is different)
PFSENSE_SSH_USER="admin"
PFSENSE_SSH_KEY="$HOME/.ssh/id_pfsense"

# Allowed paths: "name:target_ip:port" — should succeed
ALLOWED_CHECKS=(
  "Target-to-Wazuh:10.10.20.10:1514"   # Wazuh agent -> manager
  "Target-to-Wazuh-Reg:10.10.20.10:1515"
)

# Forbidden paths: "name:target_ip:port" — should FAIL (isolation must hold)
FORBIDDEN_CHECKS=(
  "Attacker-to-Mgmt:10.10.10.1:22"
  "Attacker-to-SOC:10.10.20.10:1514"
)

LOG_DIR="./data/logs"
LOG_FILE="${LOG_DIR}/vlan-healthcheck-$(date +%Y-%m-%d).log"

# ---------------------------------------------------------------------------
# Helpers
# ---------------------------------------------------------------------------
mkdir -p "$LOG_DIR"
FAILURES=0

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

pass() { log "  ✅ PASS - $1"; }
fail() { log "  ❌ FAIL - $1"; FAILURES=$((FAILURES + 1)); }

# ---------------------------------------------------------------------------
# 1. Gateway reachability (confirms each VLAN interface is up on pfSense)
# ---------------------------------------------------------------------------
log "=== Checking VLAN gateways ==="
for vlan in "${!VLAN_GATEWAYS[@]}"; do
  ip="${VLAN_GATEWAYS[$vlan]}"
  if ping -c 2 -W 2 "$ip" > /dev/null 2>&1; then
    pass "VLAN $vlan gateway ($ip) is up"
  else
    fail "VLAN $vlan gateway ($ip) is NOT responding"
  fi
done

# ---------------------------------------------------------------------------
# 2. Allowed paths must succeed
# ---------------------------------------------------------------------------
log "=== Checking allowed cross-VLAN paths ==="
for entry in "${ALLOWED_CHECKS[@]}"; do
  name="${entry%%:*}"
  rest="${entry#*:}"
  ip="${rest%%:*}"
  port="${rest##*:}"
  if nc -z -w 3 "$ip" "$port" > /dev/null 2>&1; then
    pass "$name ($ip:$port) reachable as expected"
  else
    fail "$name ($ip:$port) should be reachable but isn't — check firewall rules"
  fi
done

# ---------------------------------------------------------------------------
# 3. Forbidden paths must FAIL (segmentation/isolation check)
# ---------------------------------------------------------------------------
log "=== Checking Attacker VLAN isolation (these must be BLOCKED) ==="
for entry in "${FORBIDDEN_CHECKS[@]}"; do
  name="${entry%%:*}"
  rest="${entry#*:}"
  ip="${rest%%:*}"
  port="${rest##*:}"
  if nc -z -w 3 "$ip" "$port" > /dev/null 2>&1; then
    fail "$name ($ip:$port) SUCCEEDED — isolation is broken, review firewall rules immediately"
  else
    pass "$name ($ip:$port) correctly blocked"
  fi
done

# ---------------------------------------------------------------------------
# 4. Suricata inline status on pfSense (via SSH)
# ---------------------------------------------------------------------------
log "=== Checking Suricata inline instances on pfSense ==="
if command -v ssh > /dev/null 2>&1; then
  SURICATA_STATUS=$(ssh -i "$PFSENSE_SSH_KEY" -o BatchMode=yes -o ConnectTimeout=5 \
    "${PFSENSE_SSH_USER}@${PFSENSE_HOST}" "ps aux | grep '[s]uricata'" 2>/dev/null)

  if [[ -n "$SURICATA_STATUS" ]]; then
    RUNNING_COUNT=$(echo "$SURICATA_STATUS" | wc -l)
    pass "Suricata has $RUNNING_COUNT instance(s) running on pfSense"
  else
    fail "No Suricata instances found running on pfSense — check Services > Suricata"
  fi
else
  fail "ssh not available on this host, skipped Suricata check"
fi

# ---------------------------------------------------------------------------
# Summary
# ---------------------------------------------------------------------------
log "=== Summary ==="
if [[ $FAILURES -eq 0 ]]; then
  log "All checks passed."
else
  log "$FAILURES check(s) failed. See above for details."
fi

exit $([[ $FAILURES -eq 0 ]] && echo 0 || echo 1)

```



Vamos con el script de chequeo diario. Va en **`/scripts/automation/vlan-healthcheck.sh`** — encaja directamente en la carpeta que ya tienes destinada a "automatización y tareas repetitivas".

Lo diseño para que corra desde un host en la VLAN de Management (con acceso SSH a pfSense) y haga tres tipos de verificación: 

- que las VLANs estén arriba
- que los caminos permitidos funcionen
- y —lo más importante para un portfolio de seguridad— que el aislamiento del Attacker VLAN **siga bloqueado** (una regresión de firewall que abra ese camino es justo el tipo de fallo silencioso que un check diario debe detectar).



Cómo está armado:

- **Sección CONFIG arriba del todo** — ahí ajustas las IPs reales de tus gateways, el usuario/clave SSH de pfSense, y qué puertos son "permitidos" vs "prohibidos". No toques el resto del script.

- **Chequeo de aislamiento (`FORBIDDEN_CHECKS`)** — esta es la parte que le da valor real de portfolio: no solo confirma que las VLANs están arriba, sino que activamente intenta romper el aislamiento del Attacker VLAN y marca **FAIL** si lo logra. Eso demuestra que entiendes que "segmentado" no es un estado fijo, es algo que hay que verificar.

- **Log diario** en `./data/logs/vlan-healthcheck-YYYY-MM-DD.log` — encaja con tu carpeta `/data` ya definida, y te da histórico para meter en la bitácora diaria si algo falla.

- **Cron sugerido** en el comentario de cabecera para correrlo automáticamente.

Requiere `nc` (netcat) y acceso SSH por clave a pfSense — si prefieres evitar SSH y hacerlo todo vía la API REST de pfSense (más "profesional" mostrar en el repo), puedo adaptarlo, pero añade una capa de configuración extra (token API). ¿Lo dejamos así con SSH o lo cambio a la API de pfSense?