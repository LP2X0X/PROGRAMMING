#!/bin/bash

# Get the current branch name
branch_name=$(git rev-parse --abbrev-ref HEAD)

# Extract the ticket ID using regex (e.g., ISW-2187)
ticket_id=$(echo "$branch_name" | grep -oE '[A-Z]+-[0-9]+')

# Path to the commit message file
commit_msg_file="$1"

# Skip if:
# - The branch name is "HEAD" (detached HEAD state)
# - The ticket ID is empty (no match found)
# - The commit message already contains the ticket ID
if [[ "$branch_name" == "HEAD" ]] || [[ -z "$ticket_id" ]] || grep -q "[$ticket_id]" "$commit_msg_file"; then
    exit 0
fi

# Prepend the ticket ID to the commit message
sed -i "1s/^/[$ticket_id] /" "$commit_msg_file"

