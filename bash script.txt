#!/bin/bash

# File: log_analysis.sh
# Description: Apache log file analysis script

LOG_FILE="access.log"

if [[ ! -f "$LOG_FILE" ]]; then
  echo "Log file not found: $LOG_FILE"
  exit 1
fi

echo "------ Log Analysis Report ------"
echo "Total number of requests:"
wc -l < "$LOG_FILE"

echo -e "\nNumber of successful requests (200):"
awk '$9 ~ /^200$/' "$LOG_FILE" | wc -l

echo -e "\nNumber of failed requests (4xx and 5xx):"
awk '$9 ~ /^[45][0-9][0-9]$/' "$LOG_FILE" | wc -l

echo -e "\nTop 10 IPs making requests:"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head

echo -e "\nTop 10 requested pages:"
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -nr | head

echo -e "\nRequests per status code:"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr

echo -e "\nTop days with highest number of failures:"
awk '$9 ~ /^[45]/ {split($4, d, ":"); gsub("\\[", "", d[1]); fails[d[1]]++} END {for (day in fails) print day, fails[day]}' "$LOG_FILE" | sort -k2 -nr | head

echo -e "\nPossible attack: IPs with too many requests ( >100 )"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | awk '$1 > 100' | sort -nr

echo "------ End of Report ------"
