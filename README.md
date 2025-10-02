# starter
# filename: add_contribution_today.sh
# Usage:
#   bash add_contribution_today.sh <github-username> <repo-name>
# Example:
#   bash add_contribution_today.sh scalewithabde my-daily-commit-repo
#
# What it does:
# - Ensures git is configured with your name/email
# - Creates repo locally if missing, or pulls latest if it exists
# - Appends a timestamp line to README.md
# - Commits and pushes to GitHub main branch
#
# Requirements:
# - Git installed
# - GitHub account & repo exists (public or private)
# - Remote "origin" is set to your GitHub repo (HTTPS or SSH)
# - Your git user.email must match an email verified on your GitHub account,
#   otherwise contributions won’t show.

set -euo pipefail

if [ $# -lt 2 ]; then
  echo "Usage: bash add_contribution_today.sh <github-username> <repo-name>"
  exit 1
fi

GITHUB_USER="$1"
REPO_NAME="$2"

# Detect if git is installed
if ! command -v git >/dev/null 2>&1; then
  echo "Error: git not installed. Please install Git and re-run."
  exit 1
fi

# Check git identity
CURRENT_NAME="$(git config --global user.name || true)"
CURRENT_EMAIL="$(git config --global user.email || true)"

if [ -z "$CURRENT_NAME" ] || [ -z "$CURRENT_EMAIL" ]; then
  echo "Git user.name or user.email not set."
  echo "Enter your Git identity (must match a verified email on GitHub):"
  read -rp "Your full name: " NAME
  read -rp "Your email (verified on GitHub): " EMAIL
  git config --global user.name "$NAME"
  git config --global user.email "$EMAIL"
  echo "Configured git user.name='$NAME', user.email='$EMAIL'"
else
  echo "Using git identity: user.name='$CURRENT_NAME', user.email='$CURRENT_EMAIL'"
fi

# Clone or enter the repo directory
if [ -d "$REPO_NAME/.git" ]; then
  echo "Repo '$REPO_NAME' found locally. Updating..."
  cd "$REPO_NAME"
  git pull --rebase || true
else
  echo "Local repo not found. Attempting to clone from GitHub..."
  # Try HTTPS first; replace with SSH if you prefer (git@github.com:<user>/<repo>.git)
  REPO_URL="https://github.com/${GITHUB_USER}/${REPO_NAME}.git"
  git clone "$REPO_URL" "$REPO_NAME"
  cd "$REPO_NAME"
fi

# Ensure main branch exists or fall back to default
DEFAULT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"
echo "Current branch: $DEFAULT_BRANCH"

# Create README if missing
if [ ! -f "README.md" ]; then
  echo "# ${REPO_NAME}" > README.md
  echo "" >> README.md
fi

# Append a timestamped line (Casablanca timezone assumed by your system clock)
NOW="$(date '+%Y-%m-%d %H:%M:%S %Z')"
echo "- Daily contribution: ${NOW}" >> README.md
echo "Appended contribution line to README.md"

# Commit and push
git add README.md
git commit -m "chore: daily contribution ${NOW}"
git push origin "$DEFAULT_BRANCH"

echo "Done! Pushed a commit to ${GITHUB_USER}/${REPO_NAME} on branch ${DEFAULT_BRANCH}."
echo "Notes:"
echo " - If this is a private repo, turn on 'Show private contributions' in GitHub > Settings > Profile."
echo " - Ensure your git user.email matches a verified email on your GitHub account."
echo " - Contributions graph can take a couple minutes to update."
