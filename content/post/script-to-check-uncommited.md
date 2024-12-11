+++
date = '2024-12-11T15:32:40+02:00'
draft = false
title = 'Script to Check Uncommited Repositories'
+++

## Introduction
This script checks all repositories in a directory for uncommitted changes. I noticed that I have a lot of repositories on my machine, and sometimes I forget to commit changes when experimenting with different environments. 

This script helps me to check all repositories at once.

```bash
#!/bin/bash

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Function to check git repository status
check_git_status() {
    local repo_path="$1"
    local current_dir=$(pwd)
    local abs_path=$(realpath "$repo_path" 2>/dev/null)

    if [ ! -d "$abs_path" ]; then
        echo -e "${RED}Cannot access: ${NC}$repo_path"
        return 1
    fi

    # Change to repo directory
    cd "$abs_path" || return 1

    # Check if there are any changes
    if ! git diff --quiet || ! git diff --cached --quiet || [ -n "$(git ls-files --others --exclude-standard)" ]; then
        echo -e "${RED}Changes found in: ${NC}$repo_path"
        echo "Status:"
        git status --short
        echo "------------------------"
        cd "$current_dir"
        return 1
    else
        echo -e "${GREEN}Clean: ${NC}$repo_path"
        cd "$current_dir"
        return 0
    fi
}

# Function to find git repositories
find_git_repos() {
    local start_path="$1"
    local abs_start_path=$(realpath "$start_path" 2>/dev/null)
    local changes_found=0

    if [ ! -d "$abs_start_path" ]; then
        echo -e "${RED}Directory not found: ${NC}$start_path"
        exit 1
    fi

    # Find all .git directories, excluding hidden directories
    while IFS= read -r repo; do
        repo_path=$(dirname "$repo")
        if ! check_git_status "$repo_path"; then
            ((changes_found++))
        fi
    done < <(find "$abs_start_path" -type d -name ".git" ! -path "*/\.*/*" 2>/dev/null)

    echo "------------------------"
    if [ $changes_found -gt 0 ]; then
        echo -e "${RED}Found $changes_found repositories with uncommitted changes${NC}"
    else
        echo -e "${GREEN}All repositories are clean${NC}"
    fi
}

# Start from current directory if no argument is provided
start_path="${1:-.}"

echo "Searching for Git repositories in: $start_path"
echo "------------------------"
find_git_repos "$start_path"
```

## Usage
You can run the script with a directory path as an argument. If no argument is provided, it will start from the current directory.

```bash
bash check_uncommitted.sh /path/to/directory
```
