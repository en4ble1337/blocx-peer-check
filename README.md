
# Docker Peer Monitoring and Restart Script

This script automatically monitors a Docker container's log output for a specific peer count. If the peer count is not equal to the desired number (in this case, 3), it gracefully restarts the Docker Compose services and creates a log entry detailing the failure.

## The Script

Here is the complete `peer_check.sh` script.

```bash
#!/bin/bash

# Navigate to the Docker Compose directory
cd /root/BLOCX-Validator-Setup || exit

# Define the log file
LOG_FILE="peer_failed_log.txt"

# Check the Docker logs for the peer count
# This pipeline gets the most recent log line with a peer count and extracts the number.
PEER_COUNT=$(docker logs blocx-validator-setup-beacon-1 2>&1 | grep -o 'peers: [0-9]*' | tail -1 | awk '{print $2}')

# Verify if the peer count is not equal to 3
if [ "$PEER_COUNT" -ne 3 ]; then
    # Append a timestamped log entry indicating failure
    echo "$(date): Peer count is $PEER_COUNT, which is not 3. Restarting the services." >> "$LOG_FILE"
    reboot
fi
```
Here is the complete with log on success (optional) `peer_check.sh` script.
```bash
#!/bin/bash

# Navigate to the Docker Compose directory
cd /root/BLOCX-Validator-Setup || exit

# Define the log files
FAIL_LOG="peer_failed_log.txt"
SUCCESS_LOG="peer_success_log.txt"

# Check the Docker logs for the peer count
PEER_COUNT=$(docker logs blocx-validator-setup-beacon-1 2>&1 | grep -o 'peers: [0-9]*' | tail -1 | awk '{print $2}')

# Verify if the peer count is not equal to 3
if [ "$PEER_COUNT" -ne 3 ]; then
    # Log the failure and restart
    echo "$(date): Peer count is $PEER_COUNT, which is not 3. Restarting services." >> "$FAIL_LOG"
    reboot
else
    # Log the success
    echo "$(date): Peer check successful. Found $PEER_COUNT peers." >> "$SUCCESS_LOG"
fi
```


-----

## ⚙️ How It Works: A Detailed Breakdown

> ```bash
> cd /root/BLOCX-Validator-Setup || exit
> ```
>
> Changes the current location to `/root/BLOCX-Validator-Setup`. This is **critical** so that the `docker compose` commands can find the correct `compose-validator.yaml` file. The `|| exit` part is a safety measure that stops the script if the directory cannot be found.

> ```bash
> PEER_COUNT=$(docker logs ... | grep ... | tail ... | awk ...)
> ```
>
> This is the core command pipeline that finds the most recent peer count:
>
> 1.  `docker logs blocx-validator-setup-beacon-1`: Gets the logs for the specified container.
> 2.  `grep -o 'peers: [0-9]*'`: Filters the logs to find and extract **only** the text that matches `peers: [some number]`.
> 3.  `tail -1`: From the list of matches, it takes only the **last one**, ensuring we have the most current status.
> 4.  `awk '{print $2}'`: From the result (e.g., `peers: 3`), it prints only the **second field**, which is the number. This number is then stored in the `PEER_COUNT` variable.

> ```bash
> if [ "$PEER_COUNT" -ne 3 ]; then
> ```
>
> This line checks if the value stored in `$PEER_COUNT` is **n**ot **e**qual (`-ne`) to `3`. The restart logic inside the `if` block will only execute if this condition is true.

> ```bash
> echo "$(date): ..." >> "$LOG_FILE"
> ```
>
> If the peer count fails the check, this command creates a timestamped entry in the `peer_failed_log.txt` file, noting the failure and the incorrect peer count found.

> ```bash
> docker compose ... down
> sleep 10
> docker compose ... up -d
> ```
>
> This sequence handles the restart:
>
> 1.  `down`: Stops and removes the containers.
> 2.  `sleep 10`: Pauses for 10 seconds.
> 3.  `up -d`: Recreates and starts the containers in the background (`-d`).

-----

## 🚀 Setup and Usage

To use this script, follow these steps.

### 1\. Save the Script

Save the code above into a file named `peer_check.sh` inside your `/root/BLOCX-Validator-Setup/` directory.

### 2\. Make It Executable

Open your terminal and run this command to give the script permission to be executed:

```bash
chmod +x /root/BLOCX-Validator-Setup/peer_check.sh
```

### 3\. Schedule the Cronjob

To run the script automatically every 24 hours, you need to add it to your crontab.

1.  Open the crontab editor:

    ```bash
    crontab -e
    ```

2.  Add the following line at the bottom of the file. This schedule will run the script every day at midnight.

    ```cron
    0 0 * * * /root/BLOCX-Validator-Setup/peer_check.sh
    ```

Save and close the file. The cronjob is now active\!
