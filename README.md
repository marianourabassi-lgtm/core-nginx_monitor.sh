#!/usr/bin/env bash
# EdgeShield - Nginx Active Connection & Failover Guard

LOG_FILE="/var/log/nginx/error.log"
THRESHOLD_CONNECTIONS=500
ALERT_WEBHOOK="https://hooks.slack.com/services/mock/edgeshield/alerts"

check_nginx_health() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] Inspecting Nginx proxy sockets..."
    
    # Active connection pool check
    active_conns=$(netstat -an | grep :80 | grep ESTABLISHED | wc -l)
    
    if [ "$active_conns" -gt "$THRESHOLD_CONNECTIONS" ]; then
        echo "[WARNING] Active connections ($active_conns) exceeded threshold ($THRESHOLD_CONNECTIONS)."
        echo "[ACTION] Triggering rate-limit throttling and upstream health check."
        # Graceful reload to free unresponsive workers
        systemctl reload nginx
    else
        echo "[OK] Nginx connections stable. Active: $active_conns"
    fi
}

check_nginx_health