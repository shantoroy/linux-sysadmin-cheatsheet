# Process Management Scenarios

## Monitoring and Controlling Processes

### Find and kill processes by name
```bash
# Find and kill all processes matching "processname"
pkill processname

# Find and kill with a more precise match
pgrep -f "exact process string" | xargs kill

# Kill processes more aggressively
pkill -9 processname
```

### Find processes consuming high resources
```bash
# Find top 5 CPU-consuming processes
ps aux --sort=-%cpu | head -6

# Find top 5 memory-consuming processes
ps aux --sort=-%mem | head -6

# Find processes using over 50% CPU
ps aux | awk '$3 > 50.0'
```

### Kill all processes of a specific user
```bash
# Kill all processes owned by a user
pkill -u username

# Kill more aggressively
pkill -9 -u username
```

### Run a process that continues after logout
```bash
# Run a command in the background that continues after logout
nohup long_running_command &

# Alternative using screen
screen -dmS sessionname command

# Alternative using tmux
tmux new-session -d 'command'
```

### Limit resource usage of a process
```bash
# Limit CPU and memory
systemd-run --user --scope -p CPUQuota=50% -p MemoryLimit=1G command

# Traditional way, limiting CPU niceness (-20 to 19, higher = less priority)
nice -n 10 command

# Limit IO priority (0-7, lower = higher priority, requires ionice command)
ionice -c2 -n7 command
```

### Monitor processes in real time
```bash
# Use top with specific sorting
top -o %CPU

# Use htop with tree view
htop -t

# Monitor specific processes by name
watch "ps aux | grep [p]rocessname"
```

### Auto-restart a crashed process
```bash
# Simple monitoring script
cat > /usr/local/bin/process_monitor.sh << 'EOF'
#!/bin/bash
while true; do
  if ! pgrep -f "process_name" > /dev/null; then
    /path/to/process_command &
    echo "Process restarted at $(date)" >> /var/log/process_monitor.log
  fi
  sleep 60
done
EOF
chmod +x /usr/local/bin/process_monitor.sh

# Run the monitor as a systemd service
cat > /etc/systemd/system/process-monitor.service << EOF
[Unit]
Description=Process Monitoring Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/process_monitor.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable process-monitor.service
systemctl start process-monitor.service
```